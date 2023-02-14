# Kartta2021

Huom. testattu 'minSdkVersion 28' määrityksellä.

Tutoriaali OpenStreetMap käyttöön omassa sovelluksessa. Huom. Kartan kaupallinen käyttö vaatii tutustumista käyttöehtoihin...

Aluksi vaadittu OSM-kirjasto `build.gradle(:app)` tiedoston depencies osaan, Ja toinen jota käytetään kartta-aineiston lataukseen:

        implementation 'org.osmdroid:osmdroid-android:6.1.10'
        implementation 'androidx.preference:preference:1.2.0'

Kartta vaatii oikeudet `AndroidManifest.xml` tiedostoon:

        <uses-permission android:name="android.permission.INTERNET" />

Käyttöliittymä `activity_main.xml` sisältää MapView -komponentin. Ei muuta.

    <org.osmdroid.views.MapView
        android:id="@+id/mapview"
        tilesource="Mapnik"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

Seuraavaksi muutoksia MainActivity.java tiedostooon. Aluksi muutama luokkamuuttuja:
    
```java
    private static final String TAG="softa_kartta";
    private MapView mMapView =null;
    private MapController mMapController;

    private final int REQUEST_PERMISSIONS_REQUEST_CODE = 101;
```

Ja sitten onCreate metodiin tarpeeliset osiot suunnilleen seuraavassa järjestyksessä:
* Kartta-aineiston lataus
* MapView määrittely
* Käyttöliittymään yhdistäminen
* oikeuksien kysely. Tässä esitetty tapa hyväksyy monta kyselyä samaan aikaan (requestPermissionsIfNecessary()-metodi kohta...)
* Karttapiste Hervoodiin
* Kartan keskitys pisteeseen
```java
        Context ctx = this.getApplicationContext();
        Configuration.getInstance().load(ctx, PreferenceManager.getDefaultSharedPreferences(ctx));

        mMapView = findViewById(R.id.mapview);
        mMapView.setTileSource(TileSourceFactory.MAPNIK);
        mMapController = (MapController) mMapView.getController();
        mMapController.setZoom(18);

        //kysellään oikeudet - ja tässä olisi mahdollista kysyä usemapi...
        requestPermissionsIfNecessary(new String[]{
                Manifest.permission.INTERNET
                //, Manifest.permission.ACCESS_COARSE_LOCATION
        });

        //Paikka
        GeoPoint gPt = new GeoPoint(61.44989, 23.85688);

        mMapView.getController().setCenter(gPt);
```

Oikeuksien kyselyyn metodi joka hyväksyy monta(taulukkona):

```java
    private void requestPermissionsIfNecessary(String[] permissions) {
        ArrayList<String> permissionsToRequest = new ArrayList<>();
        for (String permission : permissions) {
            if (ContextCompat.checkSelfPermission(this, permission)
                != PackageManager.PERMISSION_GRANTED) {
                permissionsToRequest.add(permission);
            }
        }
        if (permissionsToRequest.size() > 0) {
            ActivityCompat.requestPermissions(
                this,
                permissionsToRequest.toArray(new String[0]),
                REQUEST_PERMISSIONS_REQUEST_CODE);
        }
    }
```

Ja tulosten käsittely oikeuksien kyselystä

```java
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        ArrayList<String> permissionsToRequest = new ArrayList<>();
        for (int i = 0; i < grantResults.length; i++) {
            permissionsToRequest.add(permissions[i]);
        }
        if (permissionsToRequest.size() > 0) {
            ActivityCompat.requestPermissions(
                    this,
                    permissionsToRequest.toArray(new String[0]),
                    REQUEST_PERMISSIONS_REQUEST_CODE);
        }
    }
```
Ja jos ei käynnistyksessä tule karttakuvaa, niin kommenttia tänne mika.saari@tuni.fi