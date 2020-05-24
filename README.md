# backgroundLocation

&#x27;A new Flutter project. Created by Slidy&#x27;

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://flutter.dev/docs/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://flutter.dev/docs/cookbook)

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials,
samples, guidance on mobile development, and a full API reference.


 - flutter packages pub run build_runner watch --delete-conflicting-outputs

 ## Coinfiguracao 
 
    1.  Alterar android\app\build.gradle  => compileSdkVersion para 29 e targetSdkVersion para 29
    2.  Alterar android\app\build.gradle => Adicionar dependencias em dependencies...  implementation 'com.google.android.gms:play-services-location:16.0.0'
    3. Alterar onStartCommand no arquivo android\app\src\main\kotlin\com\example\backgroundLocation\LocationUpdatesService.java
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            Log.i(TAG, "Service started");
            boolean startedFromNotification = intent.getBooleanExtra(EXTRA_STARTED_FROM_NOTIFICATION,
                    false);

            context = getApplicationContext();

            flutterEngine = new FlutterEngine(this);
            flutterEngine.getNavigationChannel().setInitialRoute("/callback");
            flutterEngine.getDartExecutor().executeDartEntrypoint(DartExecutor.DartEntryPoint.createDefault());
            flutterEngine.getPlugins().add(new io.flutter.plugins.sharedpreferences.SharedPreferencesPlugin());

            channel = new MethodChannel(flutterEngine.getDartExecutor(), "geolocation_plugin")

            // We got here because the user decided to remove location updates from the notification.
            if (startedFromNotification) {
                removeLocationUpdates();
                stopSelf();
            }
            // Tells the system to not try to recreate the service after it has been killed.
            return START_NOT_STICKY;
        }
    
    4. Alterar onNewLocation no arquivo android\app\src\main\kotlin\com\example\backgroundLocation\LocationUpdatesService.java
        private void onNewLocation(Location location) {
                Log.i(TAG, "New location: " + location);

                mLocation = location;

                channel.invokeMethod("callbackLocation", location.getLatitude() + "," +  location.getLongitude()  + "," + location.getSpeed() + "," + serviceIsRunningInForeground(this));
                
                // Notify anyone listening for broadcasts about the new location.
                Intent intent = new Intent(ACTION_BROADCAST);
                intent.putExtra(EXTRA_LOCATION, location);
                LocalBroadcastManager.getInstance(getApplicationContext()).sendBroadcast(intent);

                // Update notification content if running as a foreground service.
                if (serviceIsRunningInForeground(this)) {
                    mNotificationManager.notify(NOTIFICATION_ID, getNotification());
                }
            }
    
    
    5.  Alterado texto da mensagem no arquivo android\app\src\main\kotlin\com\example\backgroundLocation\Utils.java
    static String getLocationTitle(Context context) {
        return "Estamos obtendo sua localização: " + 
                DateFormat.getDateTimeInstance().format(new Date());
    }
    
    6. Incluir  no android\app\src\main\AndroidManifest.xml
        <uses-permission android:name="android.permission.INTERNET"/>
        <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
        <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
        <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
        <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    7. Incluir no final  do android\app\src\main\AndroidManifest.xml
        <service 
          android:name=".LocationUpdatesService"
          android:enabled="true"
          android:exported="true"
          android:foregroundServiceType="location"
        />


