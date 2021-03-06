---
title: チュートリアル:Android アプリを移行する | Microsoft Azure Maps
description: Google Maps から Microsoft Azure Maps に Android アプリを移行する方法。
author: rbrundritt
ms.author: richbrun
ms.date: 12/17/2019
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 643d48cb931bcec1a8a3385d2ec24a394660c368
ms.sourcegitcommit: f9601bbccddfccddb6f577d6febf7b2b12988911
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2020
ms.locfileid: "75909183"
---
# <a name="migrate-an-android-app-from-google-maps"></a>Google Maps から Android アプリを移行する

Azure Maps Android SDK の API インターフェイスは、Web SDK とよく似ています。 これらの SDK のいずれかを使用して開発した場合、同じ概念、ベスト プラクティス、アーキテクチャの多くが適用され、知識を簡単に流用できるはずです。

Azure Maps Android SDK でサポートされている Android の最小バージョンは、API 21: Android 5.0.0 (Lollipop) です。

すべての例は Java で提供されていますが、Azure Maps Android SDK では Kotlin も使用できます。

また、この SDK を使用した開発の詳細については、[Azure Maps Android SDK のハウツー ガイド](how-to-use-android-map-control-library.md)のページも参照してください。

## <a name="load-a-map"></a>マップを読み込む

Google Maps または Azure Maps を使用する Android アプリでのマップの読み込みを構成する手順の多くは同じです。 いずれかの SDK を使うときは、次のことを行う必要があります。

- いずれかのプラットフォームにアクセスするための API キーまたはサブスクリプション キーを取得します。
- アクティビティに XML を追加して、マップのレンダリング先とレイアウト方法を指定します。
- マップ ビューを含むアクティビティから、マップ クラスの対応するものに、すべてのライフサイクル メソッドを転送します。 特に、次のメソッドを転送する必要があります。
    - `onCreate(Bundle)`
    - `onStart()`
    - `onResume()`
    - `onPause()`
    - `onStop()`
    - `onDestroy()`
    - `onSaveInstanceState(Bundle)`
    - `onLowMemory()`
- マップの準備ができるまで待ってから、プログラムでアクセスを試みます。

**前: Google Maps**

Google Maps SDK for Android を使用してマップを表示するには、次の手順のようにします。

1.  Google Play サービスがインストールされていることを確認します。
2.  Google Maps サービスの依存関係を、モジュールの **gradle.build** ファイルに追加します。 

    `implementation 'com.google.android.gms:play-services-maps:17.0.0'`

1.  **google\_maps\_api.xml** ファイルのアプリケーション セクションに、Google Maps API キーを追加します。
    
    ```xml
    <meta-data android:name="com.google.android.geo.API_KEY" android:value="YOUR_GOOGLE_MAPS_KEY"/>
    ```

1.  メイン アクティビティにマップ フラグメントを追加します。

    ```xml
    <com.google.android.gms.maps.MapView
            android:id="@+id/myMap"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
    ```

1.  **MainActivity.java** ファイルには、Google Maps SDK のインポートを追加する必要があります。 マップ ビューを含むアクティビティから、マップ クラスの対応するものに、すべてのライフサイクル メソッドを転送します。 `MapView` インスタンスは、`getMapAsync(OnMapReadyCallback)` メソッドを使用してマップ フラグメントから取得できます。 `MapView` によって、Maps システムとビューが自動的に初期化されます。 **MainActivity.java** ファイルを次のように編集します。

    ```java
    import com.google.android.gms.maps.GoogleMap;
    import com.google.android.gms.maps.MapView;
    import com.google.android.gms.maps.OnMapReadyCallback;
    
    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    
    public class MainActivity extends AppCompatActivity implements OnMapReadyCallback {
    
        MapView mapView;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            mapView = findViewById(R.id.myMap);
    
            mapView.onCreate(savedInstanceState);
            mapView.getMapAsync(this);
        }
    
        @Override
    
        public void onMapReady(GoogleMap map) {
            //Add your post map load code here.
        }
    
        @Override
        public void onResume() {
            super.onResume();
            mapView.onResume();
        }
    
        @Override
        protected void onStart(){
            super.onStart();
            mapView.onStart();
        }
    
        @Override
        public void onPause() {
            super.onPause();
            mapView.onPause();
        }
    
        @Override
        public void onStop() {
            super.onStop();
            mapView.onStop();
        }
    
        @Override
        public void onLowMemory() {
            super.onLowMemory();
            mapView.onLowMemory();
        }
    
        @Override
        protected void onDestroy() {
            super.onDestroy();
            mapView.onDestroy();
        }
    
        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            mapView.onSaveInstanceState(outState);
        }
    }
    ```

アプリケーションで実行されると、マップ コントロールが次のように読み込まれます。

<center>

![シンプルな Google Maps](media/migrate-google-maps-android-app/simple-google-maps.png)</center>

**後: Azure Maps**

Azure Maps SDK for Android を使用してマップを表示するには、次の手順のようにします。

1. 最上位の **build.gradle** ファイルを開き、次のコードを**すべてのプロジェクト**の**リポジトリ** ブロック セクションに追加します。

    ```
    maven {
            url "https://atlas.microsoft.com/sdk/android"
    }
    ```

2. **app/build.gradle** を更新し、以下のコードを追加します。
    
    1. プロジェクトの **minSdkVersion** が API 21 以降であることを確認します。

    2. Android セクションに、次のコードを追加します。

        ```
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_8
            targetCompatibility JavaVersion.VERSION_1_8
        }
        ```
    3. 依存関係のブロックを更新し、最新の Azure Maps Android SDK の新しい実装の依存関係の行を追加します。

        ```
        implementation "com.microsoft.azure.maps:mapcontrol:0.2"
        ```

        > [!Note]
        > Azure Maps Android SDK は定期的にアップグレードされ、強化されています。 最新の Azure Maps 実装のバージョン番号は、[Android マップ コントロールの概要](how-to-use-android-map-control-library.md)ドキュメントで確認できます。 また、バージョン番号を "0.2" から "0+" に設定して、常に最新バージョンをポイントするようにできます。
    
    4. ツール バーの **[ファイル]** に移動し、 **[Sync Project with Gradle Files]\(プロジェクトを Gradle ファイルと同期\)** をクリックします。
3. メイン アクティビティにマップ フラグメントを追加します (res \> layout \> activity\_main.xml)。
    
    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >

        <com.microsoft.azure.maps.mapcontrol.MapControl
            android:id="@+id/mapcontrol"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            />
    </FrameLayout>
    ```

4. **MainActivity.java** ファイルで、次を行う必要があります。
    
    * Azure Maps SDK の imports を追加する
    * Azure Maps の認証情報を設定する
    * **onCreate** メソッドでマップ コントロールのインスタンスを取得する

    `setSubscriptionKey` または `setAadProperties` メソッドを使用して `AzureMaps` クラスにグローバルに認証情報を設定すると、すべてのビューで認証情報を追加する必要がなくなります。 

    マップ コントロールには、Android の OpenGL ライフサイクルを管理するための独自のライフサイクル メソッドが含まれており、含んでいるアクティビティから直接呼び出す必要があります。 アプリがマップ コントロールのライフサイクル メソッドを正しく呼び出すためには、マップ コントロールを含むアクティビティで次のライフサイクル メソッドをオーバーライドし、それぞれのマップ コントロール メソッドを呼び出す必要があります。 

    * `onCreate(Bundle)` 
    * `onStart()` 
    * `onResume()` 
    * `onPause()` 
    * `onStop()` 
    * `onDestroy()` 
    * `onSaveInstanceState(Bundle)` 
    * `onLowMemory()`

    **MainActivity.java** ファイルを次のように編集します。
    
    ```java
    package com.example.myapplication;

    import android.support.v7.app.AppCompatActivity;
    import android.os.Bundle;
    import com.microsoft.azure.maps.mapcontrol.AzureMaps;
    import com.microsoft.azure.maps.mapcontrol.MapControl;
    import com.microsoft.azure.maps.mapcontrol.layer.SymbolLayer;
    import com.microsoft.azure.maps.mapcontrol.options.MapStyle;
    import com.microsoft.azure.maps.mapcontrol.source.DataSource;

    public class MainActivity extends AppCompatActivity {
        
        static {
            AzureMaps.setSubscriptionKey("<Your Azure Maps subscription key>");
        }

        MapControl mapControl;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            mapControl = findViewById(R.id.mapcontrol);

            mapControl.onCreate(savedInstanceState);
    
            //Wait until the map resources are ready.
            mapControl.onReady(map -> {
                //Add your post map load code here.
    
            });
        }

        @Override
        public void onResume() {
            super.onResume();
            mapControl.onResume();
        }

        @Override
        protected void onStart(){
            super.onStart();
            mapControl.onStart();
        }

        @Override
        public void onPause() {
            super.onPause();
            mapControl.onPause();
        }

        @Override
        public void onStop() {
            super.onStop();
            mapControl.onStop();
        }

        @Override
        public void onLowMemory() {
            super.onLowMemory();
            mapControl.onLowMemory();
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            mapControl.onDestroy();
        }

        @Override
        protected void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            mapControl.onSaveInstanceState(outState);
        }
    }
    ```

アプリケーションを実行すると、マップ コントロールが次のように読み込まれます。

<center>

![シンプルな Azure Maps](media/migrate-google-maps-android-app/simple-azure-maps.png)</center>

Azure Maps コントロールではさらに縮小がサポートされていて、より多くのワールド ビューが提供されることに注意してください。

> [!TIP]
> Windows で Android エミュレーターを使用している場合、OpenGL およびソフトウェア アクセラレータのグラフィックス レンダリングとの競合が原因で、マップがレンダリングされないことがあります。 この問題を解決するには、次のようにします。 AVD マネージャーを開き、編集する仮想デバイスを選択します。 **[Emulated Performance]\(エミュレートされたパフォーマンス\)** セクションで、 **[Graphics]\(グラフィックス\)** オプションを **[Hardware]\(ハードウェア\)** に設定します。

## <a name="localizing-the-map"></a>マップのローカライズ

対象ユーザーが複数の国に分散している場合や、使用されている言語が異なる場合は、ローカライズが重要になります。

**前: Google Maps**

マップの言語は、マップのコンテキスト ビューを設定する前に次のコードを追加することによって、メイン アクティビティの `onCreate` メソッドで設定できます。 次の例では、言語コード "fr" を使用して言語をフランス語に限定しています。

```java
String languageToLoad = "fr";
Locale locale = new Locale(languageToLoad);
Locale.setDefault(locale);
Configuration config = new Configuration();
config.locale = locale;
getBaseContext().getResources().updateConfiguration(config,
        getBaseContext().getResources().getDisplayMetrics());
```

言語が "fr" に設定されている Google Maps の例を以下に示します。

<center>

![Google Maps のローカライズ](media/migrate-google-maps-android-app/google-maps-localization.png)</center>

**後: Azure Maps**

Azure Maps には、マップの言語と地域ビューを設定するための 3 つの異なる方法が用意されています。 最初のオプションでは、静的な `setLanguage` メソッドと `setView` メソッドをグローバルに使用して、言語と地域ビューの情報を `AzureMaps` クラスに渡します。 これにより、アプリに読み込まれたすべての Azure Maps コントロールに対して、既定の言語と地域ビューが設定されます。 次の例では、言語コード "fr-FR" を使用して言語をフランス語に限定しています。

```java
static {
    //Set your Azure Maps Key.
    AzureMaps.setSubscriptionKey("<Your Azure Maps Key>");

    //Set the language to be used by Azure Maps.
    AzureMaps.setLanguage("fr-FR");

    //Set the regional view to be used by Azure Maps.
    AzureMaps.setView("auto");
}
```

2 つ目のオプションでは、言語とビュー情報をマップ コントロール XML に渡します。

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_language="fr-FR"
    app:mapcontrol_view="auto"
    />
```

3 つ目のオプションでは、マップの `setStyle` メソッドを使用して、マップの言語と地域ビューをプログラムで設定します。 これは、マップの言語と地域ビューを変更するときにいつでも実行できます。

```java
mapControl.onReady(map -> {
    map.setStyle(StyleOptions.language("fr-FR"));
    map.setStyle(StyleOptions.view("auto"));
});
```

以下に、言語が "fr-FR" に設定されている Azure Maps の例を示します。

<center>

![Azure Maps のローカライズ](media/migrate-google-maps-android-app/azure-maps-localization.png)</center>

サポートされている言語と地域ビューの完全な一覧については、[こちら](supported-languages.md)をご覧ください。

## <a name="setting-the-map-view"></a>マップ ビューの設定

Azure および Google Maps の両方の動的マップは、適切なメソッドを呼び出すことによって、プログラムで新しい地理的な場所に移動できます。 以下の例では、マップに衛星航空画像を表示し、マップの中心をある場所に指定する方法を示しています。これは、座標の緯度が 35.0272、経度が -111.0225 で、ズーム レベルを 15 に変更する Google Maps の場合です。

> [!NOTE]
> Google Maps では、ディメンションが 256 ピクセルのタイルが使用されますが、Azure Maps では、より大きな 512 ピクセルのタイルが使用されます。 これにより、Azure Maps で Google Maps と同じマップ領域を読み込むために必要なネットワーク要求の数が減ります。 しかし、マップ コントロールでのタイル ピラミッドの動作方法により、Azure Maps のより大きなタイルは、Google Maps のマップと同じ可視領域を得るために、Azure Maps の使用時に Google Maps で利用されるズーム レベルを 1 ずつ減算する必要があることを意味します。

**前: Google Maps**

Google Maps マップ コントロールのカメラは、`moveCamera` メソッドを使用してプログラムで移動できます。これにより、マップの中心とズーム レベルを指定できます。 `setMapType` メソッドを使用して、表示されるマップの種類を変更できます。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.moveCamera(CameraUpdateFactory.newLatLngZoom(new LatLng(35.0272, -111.0225), 15));
    mapView.setMapType(GoogleMap.MAP_TYPE_SATELLITE);
}
```

<center>

![Google Maps の設定ビュー](media/migrate-google-maps-android-app/google-maps-set-view.png)</center>

**後: Azure Maps**

前に説明したように、Azure Maps で同じ表示可能領域を実現するには、Google Maps で使用されているズーム レベルを 1 だけ小さくします。この場合は、ズーム レベル 14 を使用します。

初期マップ ビューは、マップ コントロールの XML 属性で設定できます。

```xml
<com.microsoft.azure.maps.mapcontrol.MapControl
    android:id="@+id/myMap"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:mapcontrol_cameraLat="35.0272"
    app:mapcontrol_cameraLng="-111.0225"
    app:mapcontrol_zoom="14"
    app:mapcontrol_style="satellite"
    />
```

マップ ビューは、マップの `setCamera` メソッドと `setStyle` メソッドを使用して、プログラムで更新できます。

```java
mapControl.onReady(map -> {
    //Set the camera of the map.
    map.setCamera(center(35.0272, -111.0225), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE));
});
```

<center>

![Azure Maps の設定ビュー](media/migrate-google-maps-android-app/azure-maps-set-view.png)</center>

**その他のリソース:**

- [サポートされているマップ スタイル](supported-map-styles.md)

## <a name="adding-a-marker"></a>マーカーの追加

マップ上のイメージを使用してマップにポイント データをレンダリングすることがよくあります。 これらのイメージは、多くの場合、マーカー、画鋲、ピン、またはシンボルと呼ばれます。 次の例では、ポイント データをマップ上のマーカーとして表示します (緯度: 51.5、経度: -0.2)。

**前: Google Maps**

Google Maps では、マップの `addMarker` メソッドを使用してマーカーを追加します。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33)));
}
```

<center>

![Google Maps のマーカー](media/migrate-google-maps-android-app/google-maps-marker.png)</center>

**後: Azure Maps**

Azure Maps では、最初にデータをデータ ソースに追加してから、そのデータ ソースをシンボル レイヤーに接続することで、ポイント データをマップに表示できます。 データ ソースによりマップ コントロールでの空間データの管理が最適化され、シンボル レイヤーにより、イメージやテキストを使用してポイント データをレンダリングする方法が指定されます。

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Create a symbol layer and add it to the map.
    map.layers.add(new SymbolLayer(dataSource));
});
```

<center>

![Azure Maps のマーカー](media/migrate-google-maps-android-app/azure-maps-marker.png)</center>

## <a name="adding-a-custom-marker"></a>カスタム マーカーの追加

カスタム イメージを使用すると、マップ上のポイントを表すことができます。 以下のイメージ例では、カスタム イメージを使用してマップ上のポイントを表示します。ここでは、緯度が 51.5、経度が -0.2 で、マーカーの位置をオフセットして、画鋲アイコンのポイントがマップ上の正しい位置に揃うようにします。

<center>

![黄色の画鋲のイメージ](media/migrate-google-maps-web-app/ylw_pushpin.png)<br/>
ylw\_pushpin.png</center>

どちらの例でも、上記のイメージはアプリ リソースの drawable フォルダーに追加されます。

**前: Google Maps**

Google Maps では、マーカーの `icon` オプションを使用して読み込むことにより、カスタム イメージをマーカーに使用できます。 イメージのポイントを座標に合わせるには、`anchor` オプションを使用します。 アンカーは、イメージの大きさに対する相対値です。この場合は、幅が 0.2 単位で、高さが 1 単位です。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.addMarker(new MarkerOptions().position(new LatLng(47.64, -122.33))
    .icon(BitmapDescriptorFactory.fromResource(R.drawable.ylw_pushpin))
    .anchor(0.2f, 1f));
}
```

<center>

![Google Maps のカスタム マーカー](media/migrate-google-maps-android-app/google-maps-custom-marker.png)</center>

**後: Azure Maps**

Azure Maps のシンボル レイヤーでもカスタム イメージがサポートされますが、最初にイメージをマップ リソースに読み込んで、一意の ID を割り当てる必要があります。 その後、シンボル レイヤーでこの ID を参照できます。 `iconOffset` オプションを使用して、イメージ上の正しいポイントに合わせるために、シンボルをオフセットできます。 アイコンのオフセットはピクセル単位であることに注意してください。 既定では、オフセットの基準はイメージの下端中央になりますが、これは `iconAnchor` オプションを使用して調整できます。 この例では、`iconAnchor` オプションを `"center"` に設定し、アイコン オフセットを使用して、イメージを右に 5 ピクセル、上に 15 ピクセルだけ移動して、画鋲のイメージの位置に合わせています。

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create a point feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64)));

    //Load the custom image icon into the map resources.
    map.images.add("my-yellow-pin", R.drawable.ylw_pushpin);

    //Create a symbol that uses the custom image icon and add it to the map.
    map.layers.add(new SymbolLayer(dataSource,
        iconImage("my-yellow-pin"),
        iconAnchor(AnchorType.CENTER
        iconOffset(new Float[]{5f, -15f})));
});
```

<center>

![Azure Maps のカスタム マーカー](media/migrate-google-maps-android-app/azure-maps-custom-marker.png)</center>

## <a name="adding-a-polyline"></a>ポリラインの追加

ポリラインは、マップ上の線またはパスを表すために使用されます。 次の例では、マップ上に破線のポリラインを作成する方法を示します。

**前: Google Maps**

Google Maps では、`PolylineOptions` クラスを使用してポリラインを作成し、`addPolyline` メソッドを使用してマップにポリラインを追加できます。 ストロークの色は `color` オプション、ストロークの幅は width オプション、ストロークの破線の配列は `pattern` オプションを使用して、設定できます。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polyline.
    PolylineOptions lineOptions = new PolylineOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .color(Color.RED)
            .width(10f)
            .pattern(Arrays.<PatternItem>asList(
                    new Dash(30f), new Gap(30f)));

    //Add the polyline to the map.
    Polyline polyline = mapView.addPolyline(lineOptions);
}
```

<center>

![Google Maps のポリライン](media/migrate-google-maps-android-app/google-maps-polyline.png)</center>

**後: Azure Maps**

Azure Maps では、ポリラインを LineString または MultiLineString オブジェクトと呼びます。 これらのオブジェクトは、データ ソースに追加し、線レイヤーを使用してレンダリングできます。 ストロークの幅と破線の配列は、Azure Maps Web SDK に合わせて "ピクセル" 単位であることに注意してください。このようにすると、どちらの SDK でも、同じ値を使用して同じ結果が得られます。

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of points.
    List<Point> points = Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46));

    //Create a LineString feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(LineString.fromLngLats(points)));

    //Create a line layer and add it to the map.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(4f),
        strokeDashArray(new Float[]{3f, 3f})));
});
```

<center>

![Azure Maps のポリライン](media/migrate-google-maps-android-app/azure-maps-polyline.png)</center>

## <a name="adding-a-polygon"></a>多角形の追加

多角形は、マップ上の領域を表すために使用されます。 次の例は、マップの中心座標に基づいて、三角形を形成する多角形を作成する方法を示しています。

**前: Google Maps**

Google Maps では、`PolygonOptions` クラスを使用して多角形を作成し、`addPolygon` メソッドを使用してマップに多角形を追加できます。 塗りつぶしとストロークの色は `fillColor` および `strokeColor` オプションを使用して設定でき、ストロークの幅は `strokeWidth` オプションを使用して設定します。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the polygon.
    PolygonOptions polygonOptions = new PolygonOptions()
            .add(new LatLng(46, -123))
            .add(new LatLng(49, -122))
            .add(new LatLng(46, -121))
            .add(new LatLng(46, -123))  //Close the polygon.
            .fillColor(Color.argb(128, 0, 128, 0))
            .strokeColor(Color.RED)
            .strokeWidth(5f);

    //Add the polygon to the map.
    Polygon polygon = mapView.addPolygon(polygonOptions);
}
```

<center>

![Google Maps の多角形](media/migrate-google-maps-android-app/google-maps-polygon.png)</center>

**後: Azure Maps**

Azure Maps では、Polygon および MultiPolygon オブジェクトをデータ ソースに追加し、レイヤーを使用してマップ上にレンダリングすることができます。 多角形の領域は、多角形レイヤーでレンダリングできます。 多角形の枠線は、線レイヤーを使用してレンダリングできます。 ストロークの幅と破線の配列は、Azure Maps Web SDK に合わせて "ピクセル" 単位であることに注意してください。このようにすると、どちらの SDK でも、同じ値を使用して同じ結果が得られます。

```java
mapControl.onReady(map -> {
    //Create a data source and add it to the map.
    DataSource dataSource = new DataSource();
    map.sources.add(dataSource);

    //Create an array of point arrays to create polygon rings.
    List<List<Point>> rings = Arrays.asList(Arrays.asList(
        Point.fromLngLat(-123, 46),
        Point.fromLngLat(-122, 49),
        Point.fromLngLat(-121, 46),
        Point.fromLngLat(-123, 46)));

    //Create a Polygon feature and add it to the data source.
    dataSource.add(Feature.fromGeometry(Polygon.fromLngLats(rings)));

    //Add a polygon layer for rendering the polygon area.
    map.layers.add(new PolygonLayer(dataSource,
        fillColor("green"),
        fillOpacity(0.5f)));

    //Add a line layer for rendering the polygon outline.
    map.layers.add(new LineLayer(dataSource,
        strokeColor("red"),
        strokeWidth(2f)));
});
```

<center>

![Azure Maps の多角形](media/migrate-google-maps-android-app/azure-maps-polygon.png)</center>

## <a name="overlay-a-tile-layer"></a>タイル レイヤーをオーバーレイする

タイル レイヤー (Google Maps ではイメージ オーバーレイとも呼ばれる) を使用すると、マップ タイリング システムに合わせて、より小さなタイル表示されたイメージに分割されたレイヤー イメージをオーバーレイすることができます。 これは、レイヤー イメージや非常に大きなデータ セットをオーバーレイする一般的な方法です。

次の例では、アイオワ州立大学の Iowa Environmental Mesonet の気象レーダー タイル レイヤーをオーバーレイします。 タイルのサイズは 256 ピクセルです。

**前: Google Maps**

Google Maps では、`TileOverlayOptions` クラスを使用してタイル レイヤーをマップの上にオーバーレイでき、`addTileLauer` メソッドを使用してマップに追加することができます。 タイルを半透明にするには、`transparency` オプションを 0.2 つまり 20% の透明度に設定します。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    //Create the options for the tile layer.
    TileOverlayOptions tileLayer = new TileOverlayOptions()
        .tileProvider(new UrlTileProvider(256, 256) {
            @Override
            public URL getTileUrl(int x, int y, int zoom) {

                try {
                    //Define the URL pattern for the tile images.
                    return new URL(String.format("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/%d/%d/%d.png", zoom, x, y));
                }catch (MalformedURLException e) {
                    throw new AssertionError(e);
                }
            }
        }).transparency(0.2f);

    //Add the tile layer to the map.
    mapView.addTileOverlay(tileLayer);
}
```

<center>

![Google Maps のタイル レイヤー](media/migrate-google-maps-android-app/google-maps-tile-layer.png)</center>

**後: Azure Maps**

Azure Maps では、他のレイヤーとほぼ同じように、タイル レイヤーをマップに追加できます。 x、y、ズーム プレースホルダー(`{x}`、`{y}`、`{z}`) の書式設定された URL はそれぞれ、タイルにアクセスする場所をレイヤーに指示するために使用されます。 Azure Maps のタイル レイヤーでは、`{quadkey}`、`{bbox-epsg-3857}`、`{subdomain}` のプレースホルダーもサポートされます。 タイル レイヤーを半透明にするには、不透明度の値 0.8 を使用します。 不透明度と透明度は似ていますが、逆の値を使用することに注意してください。 これらの値を変換するには、単にその値を 1 から減算します。

> [!TIP]
> Azure Maps では、レイヤーは、基本マップ レイヤーを含む、他のレイヤーの下に簡単にレンダリングできます。 多くの場合、読みやすいようにタイル レイヤーをマップ ラベルの下にレンダリングすることをお勧めします。 `map.layers.add` メソッドでは、2 番目のパラメーターを受け取ります。これは、下に新しいレイヤーを挿入するレイヤーの id です。 マップ ラベルの下にタイル レイヤーを挿入する場合は、次のコードを使用できます: `map.layers.add(myTileLayer, "labels");`

```java
mapControl.onReady(map -> {
    //Add a tile layer to the map, below the map labels.
    map.layers.add(new TileLayer(
        tileUrl("https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png"),
        opacity(0.8f),
        tileSize(256)
    ), "labels");
});
```

<center>

![Azure Maps のタイル レイヤー](media/migrate-google-maps-android-app/azure-maps-tile-layer.png)</center>

## <a name="show-traffic"></a>交通情報を表示する

交通データは、Azure および Google Maps の両方にオーバーレイできます。

**前: Google Maps**

Google Maps では、マップの `setTrafficEnabled` メソッドに true を渡すことで、交通量データをマップの上にオーバーレイできます。

```java
@Override
public void onMapReady(GoogleMap googleMap) {
    mapView = googleMap;

    mapView.setTrafficEnabled(true);
}
```

<center>

![Google Maps の交通情報](media/migrate-google-maps-android-app/google-maps-traffic.png)</center>

**後: Azure Maps**

Azure Maps では、交通情報を表示するためのさまざまなオプションが提供されます。 道路の閉鎖や事故などの交通事故は、マップ上にアイコンとして表示できます。 交通流量 (色分けされた道路) をマップ上にオーバーレイでき、ポストされた速度制限、通常の予想される遅延、または絶対遅延を基準として、色を変更することができます。 Azure Maps の事故データは 1 分ごとに、流量データは 2 分ごとに更新されます。

```java
mapControl.onReady(map -> {
    map.setTraffic(
        incidents(true),
        flow(TrafficFlow.RELATIVE));
});
```

<center>

![Azure Maps の交通情報](media/migrate-google-maps-android-app/azure-maps-traffic.png)</center>

## <a name="next-steps"></a>次のステップ

Azure Maps Android SDK の詳細について学習します。

> [!div class="nextstepaction"]
> [Android マップ コントロールの使用方法](how-to-use-android-map-control-library.md)
