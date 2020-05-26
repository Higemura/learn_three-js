learning-threejs
================

All examples for chapter 1.

## 3Dオブジェクトの表示
|オブジェクト|説明|
|plane|2次元の長方形で地面に相当する|
|cube|3次元の立方体で赤色で描画される|
|sphere|3次元の球体で青色で描画される|
|camera|カメラは画面には何も描画されないが、何を出力に含めるかを決定する|
|axes|x, y, z軸。これは便利なデバッグ用のツールでオブジェクトが3D空間のどこに描画されているかを確認できる。xは赤色、y軸は緑色、z軸は青色でそれぞれ描画される|


### シーンを追加
```
var scene = new THREE.Scene()
```

### カメラを追加
```
var camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
```

### レンダラーを追加
```
var renderer = new THREE.WebGLRenderer();
```

### 背景色を変更
```
// 0xEEEEEE（白）
renderer.setClearColor(new THREE.Color(0xEEEEEE));
```

### 描画するシーンの大きさを設定
```
renderer.setSize(window.innerWidth, window.innerHeight);
```

### 座標軸を追加
```
var axes = new THREE.AxisHelper(20);
scene.add(axes);
```

### 平面を定義
```
var planeGeometry = new THREE.PlaneGeometry(60, 20);
```

### 平面のマテリアル（見た目）を定義
```
// 0xcccccc（赤）
var planeMaterial = new THREE.MeshBasicMaterial({color: 0xcccccc});
```

### Meshオブジェクトを作成する
```
 var plane = new THREE.Mesh(planeGeometry, planeMaterial);
```



## マテリアル、ライト、影の追加

### 光源の追加
```
// 光源を追加
var spotLight = new THREE.SpotLight(0xffffff);
// THREE.SpotLightは自分の位置（spotLight.position.set）からシーンを照らします。
spotLight.position.set(-20, 30, -5);
```

#### マテリアルに光源を反映させるには
オブジェクトのマテリアルが`THREE.MeshBasicMaterial`だと光源に照らされてもマテリアルに反映されません。
`THREE.MeshLambertMaterial`で光源がマテリアルに反映されます。
```
 var sphereMaterial = new THREE.MeshLambertMaterial({color: 0x7777ff});
```
他にも`MeshPhongMaterial`と`MeshStandardMaterial`が光源を計算に含める事ができる。

### 影の追加
影の描画は大きな計算コストがかかるため、Three.jsではデフォルトで無効化されています。
影の描画をするにはrendererに影を描画したいと伝える必要があります。
shadowMap.enabledをtrueにする事で影の描画を許可します、
```
renderer.setClearColor(new THREE.Color(0xEEEEEE));
renderer.setSize(window.innerWidth, window.innerHeight);
// 影の描画を許可する
renderer.shadowMap.enabled = true;
```

### 影を指定する
shadowMap.enabledをtrueにしても影の変化はありません。
なぜなら**影を落とす物体**と**影を落とされる物体**がどれかを明示的に指定する必要があるからです。
平面やオブジェクトに影を描画する場合は`receiveShadow`と`castShadow`を`true`にします。

```
// plane（平面）に影を描画する
plane.receiveShadow = true;

// cube（立方体）に影を描画する
cube.castShadow = true;

// sphere（球体）に影を描画する
sphere.castShadow = true;
```

### シーンに影を追加する
影を描画するためには**シーン内のどのライトを影の発生元とするかを指定しなければなりません。**
しかし、**全てのライトが影を落とせるわけではありません。**
(THREE.SpotLightは影を落とす事が可能)
```
// 光源に影を追加する事を許可する
spotLight.castShadow = true;
```


## シーンにアニメーションを追加する
HTML5とそれに関連するJavaScript APIガ登場するまではsetInterval関数を使用するしかありませんでした。
今でもsetInterval関数でアニメーションを実装する事ができるが、以下のような問題があ李ます。
- ブラウザがどのような状態にあるか考慮しない
- 別タブを閲覧中でもお構いなしに関数が数ミリ秒ごとに実行される
- 実行タイミングが画面の再描画と同期されない
結果CPUの使用率が高くなり、パフォーマンスが著しく下がります。

### requestAnimationFrameの導入
`requestAnimationFrame`を使用するとブラウザによって定義された間隔で呼び出される関数が設定でき、必要なものをその関数ないで自由に描画できます。

```
document.getElementById("WebGL-output").appendChild(renderer.domElement);

// renderer.render関数を呼び出す代わりに一度renderScene関数を呼び出してアニメーションを開始する
renderScene();

function renderScene() {
  // アニメーション部分
  requestAnimationFrame(renderScene);

  // アニメーションを実行した後にレンダラーを実行
  renderer.render(scene, camera);
}
```

### アニメーションのフレームレートを表示する
https://github.com/mrdoob/stats.js
上記URLからstats.jsをダウンロードして<head>要素の中でライブラリを読み込む
```
<head>
  <script type="text/javascript" src="../libs/three.js"></script>
  <script type="text/javascript" src="../libs/stats.js"></script>
</head>
```
さらにグラフの出力先として使用する<div>要素を追加する
```
<div id="Stats-output"></div>
```
次に統計情報を初期化してそれを<div>要素に追加します。
```
function initStats() {
    var stats = new Stats();
    stats.setMode(0); // 0: fps, 1: ms

    // 計測情報をどこに表示するか（例ではtop-left）
    stats.domElement.style.position = 'absolute';
    stats.domElement.style.left = '0px';
    stats.domElement.style.top = '0px';

    document.getElementById("Stats-output").appendChild(stats.domElement);

    return stats;
}
```

### 立方体を回転
```
function renderScene() {
  stats.update();
  // 立方体の回転
  cube.rotation.x += 0.02;
  cube.rotation.y += 0.02;
  cube.rotation.z += 0.02;

  ...etc

  // render using requestAnimationFrame
  requestAnimationFrame(renderScene);
  renderer.render(scene, camera);
}
```

### 球体の移動
```
function renderScene() {
  stats.update();

  ...etc

  // 球体の弾む速さを定義する
  step += 0.04;
  // 三角関数
  sphere.position.x = 20 + ( 10 * (Math.cos(step)));
  sphere.position.y = 2 + ( 10 * Math.abs(Math.sin(step)));

  // render using requestAnimationFrame
  requestAnimationFrame(renderScene);
  renderer.render(scene, camera);
}
```


### 実験をもっと簡単にするためdat.GUIを使用
Googleエンジニアが開発したdat.GUIというライブラリ。
このライブラリを使用するとコード内の変数の値を変更するための、単純なユーザーインターフェースコンポーネントを簡単に作成できる。
https://github.com/dataarts/dat.gui
<head>要素にライブラリを読み込ませる
```
<head>
  <script type="text/javascript" src="../libs/dat.gui.js"></script>
</head>
```

### dat.GUIを設置する
```
// dat.GUIによって変更されるプロパティの値を保持するオブジェクトを用意
var controls = new function () {
  this.rotationSpeed = 0.02;
  this.bouncingSpeed = 0.03;
};

// controlsで設定した値をguiに渡す
var gui = new dat.GUI();
// 0~0.5の値まで設定できるようにしている
gui.add(controls, 'rotationSpeed', 0, 0.5);
gui.add(controls, 'bouncingSpeed', 0, 0.5);
```

### アニメーションさせる値をdat.GUIから読み込む
controlsで設定した値をアニメーション関数内で読み込む
```
function render() {
  stats.update();
  // controlsで設定した値から読み込む
  cube.rotation.x += controls.rotationSpeed;
  cube.rotation.y += controls.rotationSpeed;
  cube.rotation.z += controls.rotationSpeed;

  // controlsで設定した値から読み込む
  step += controls.bouncingSpeed;
  sphere.position.x = 20 + ( 10 * (Math.cos(step)));
  sphere.position.y = 2 + ( 10 * Math.abs(Math.sin(step)));

  // render using requestAnimationFrame
  requestAnimationFrame(render);
  renderer.render(scene, camera);
}
```


### ブラウザサイズが変更されたら出力を自動的にリサイズする
現状のままだと描画サイズが固定のままなので、ブラウザ画面のリサイズによってシーンのサイズも変更できるようにする。

#### リサイズイベントリスナを設定する
```
window.addEventListener('resize', onResize, false);
```
#### リサイズした時の関数を用意する
```
// リサイズした時の処理
function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
```

#### camera, scene, rendererをinit()以外の関数からも呼び出せるように外に出す
```
// init()内にあった変数を外に出す
var camera;
var scene;
var renderer;

function init() {
  var stats = initStats();
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer();
  ...etc
}

function onResize() {
  // 外出ししてある変数（camera, scene, renderer）を使用する
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}
```

## まとめ
- Three.jsを描画するには、初めに`THREE.scene`オブジェクト（シーンオブジェクト）を作成する
- 作成したシーンオブジェクトにカメラ、ライト（光源）、描画したいオブジェクトを追加する
- 影を追加
- アニメーションを追加
- dat.GUIで制御用のインターフェースを作成できる
- stats.jsでシーンが描画されるフレームレートや描画時間を表示できる