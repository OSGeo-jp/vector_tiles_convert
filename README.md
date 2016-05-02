# 基盤地図情報等ベクトルタイル変換プログラム

## プログラム概要
基盤地図情報、数値地図（国土基本情報）等のデータをベクトルタイルに変換するプログラム。

具体的な機能は次のとおり。  
1. 基盤地図情報基本項目（2次メッシュ単位にZIP化されたGML形式のデータ）をベクトルタイル（GeoJSON Tiles）に変換  
2. 基盤地図情報数値標高モデル（2次メッシュ単位にZIP化されたGML形式のデータ）をベクトルタイル（GeoJSON Tiles）に変換  
3. 数値地図（国土基本情報）地図情報（2次メッシュ単位にZIP化されたGML形式又はShapefile形式のデータ）をベクトルタイル（GeoJSON Tiles）に変換  
4. 数値地図（国土基本情報20万）（1次メッシュ単位にZIP化されたGML形式又はShapefile形式のデータ）をベクトルタイル（GeoJSON Tiles）に変換  
5. Shapefile形式のデータをベクトルタイル（GeoJson Tiles）に変換

なお、基盤地図情報、数値地図（国土基本情報）のベクトルタイル変換にあたっては、更新されたメッシュ単位での変換範囲の指定が可能である。


## プログラム構成
本プログラムは、次の4つのファイルから構成されている。
これらを同一ディレクトリに保存して実行する。
- main.py
- load.py
- clipping.py
- util.py


## プログラム実行環境設定
本プログラムは、Python2.7で記述され、様々なオープンソースライブラリを利用している。  
使用ライブラリは、Pythonのライブラリ管理ツールpipで管理できるライブラリと、そうでないものとに分けられる。  
CentOS（RedHat系 Linux）の環境を想定し、環境設定の方法を以下にまとめる。

### Pythonパッケージライブラリ
- setuptools  
  Pythonの拡張ライブラリの代表的なものは、パッケージ管理ツール「pip」によって管理される。
  - URL：https://pypi.python.org/pypi/setuptools
  - インストールの実行：`python setup.py install`
- lxml  
  xmlに関するツールキットで、Pythonの標準ライブラリである「xml」よりも機能が豊富である。
  - URL：https://pypi.python.org/pypi/lxml/3.5.0 または http://lxml.de/
  - インストールの実行：`pip install lxml`
- more_itertools  
  Pythonが標準で提供しているイテレータに関するライブラリ「itertools」を拡張し、より利便性の高いイテレータの生成が可能なライブラリ。
  - URL：https://pypi.python.org/pypi/more-itertools/
  - インストールの実行：`pip install more_itertools`
- pyproj  
  地図座標の投影や変換に関するライブラリである「PROJ.4」（ライセンス：MIT）のPythonインタフェースを提供するライブラリ。  
  タイル作成時、経緯度からタイル座標への変換に使用。
  - URL：https://pypi.python.org/pypi/pyproj/
  - インストールの実行：`pip install pyproj`
- shapely  
  C++のオープンソースジオメトリエンジンである「GEOS」（ライセンス：LGPL）を用いたPythonのジオメトリライブラリであり、ポイント、ライン、ポリゴンなどの地物に関する処理が可能。  
  タイル作成時、地物のクリップ処理に使用。
  - URL：https://pypi.python.org/pypi/Shapely/1.5.13
  - インストールの実行：`yum install geos`（GEOSがインストールされていない場合）  
  　　　　　　　　　　`pip install shapely`
- geojson  
  GeoJSONのオブジェクトをPythonで扱えるような機能をもつライブラリ。
  - URL：https://pypi.python.org/pypi/geojson/1.3.2
  - インストールの実行：`pip install geojson`
- rtree  
  空間探索のライブラリである「libspatialindex」をPythonで扱えるようにするラッパープログラム。  
  タイルのクリップ処理の際に対象地物を選定するために使用。
  - URL：https://pypi.python.org/pypi/Rtree/
  - インストールの実行：`yum install spatialindex`  
  　　　　　　　　　　`pip install rtree`

### その他のライブラリ
- globalmaptiles  
  TMS（Tile Map Service）に基づくタイルの境界座標や、タイルIDなどの算出が可能なPythonライブラリ。
  - URL1：http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/
  - URL2：https://gist.github.com/unorthodox123/5944793
  - 上記URLからファイルをダウンロードし、プログラムと同じディレクトリに配置。


## プログラムの実行方法
プログラムは、main.pyが実行ファイルになっており、ADD, MERGE, UPDATE の3種類の実行モードを有する。  
各モード別に実行方法を示す。

### ADDモード
ADDモードは、入力ファイルからベクトルタイル（メッシュ単位）を作成するモードである。  
ADDモードで指定できる引数及びオプションは次のとおり。
- path：入力ファイルのパス（メッシュ単位のZIPファイルのパス）
- base：出力するベクトルタイル（メッシュ単位）のルートディレクトリを指定する。なにも指定しない場合は、カレントディレクトリが設定される。
- code：入力ファイルの地域メッシュコード（基盤地図情報等のファイル名規則にのっとっている場合は、指定しなくてもよい）
- feature：変換の対象とするデータ種別をカンマ区切りで指定する。何も指定しない場合は、全て指定された状態になる。
- zoom：作成するベクトルタイル（メッシュ単位）のズームレベルを指定する。なにも指定しない場合は、18となる。
- s：入力ファイルがシェープファイルの場合は「-s」オプションを付ける。

### MERGEモード
MERGEモードは、ADDモードによって作成されているベクトルタイル（メッシュ単位）の全てのメッシュについてファイルを読み込み、ベクトルタイルを生成するモードである。  
従って、MERGEモードでは、ベクトルタイル（メッシュ単位）を入力とし、ベクトルタイルを出力とする。  
MERGEモードで指定できる引数及びオプションは次のとおり。

- mergedir：マージを区別する名称。mergedirで指定した文字列の頭に「M_」を付けたディレクトリがbase以下に作成され、マージ結果（ベクトルタイル）が保存される。
- base：ベクトルタイル（メッシュ単位）のルートディレクトリを指定する。なにも指定しない場合は、カレントディレクトリが設定される。ADDモードで指定していたbaseと同じ値を指定する。
- feature：変換の対象とするデータ種別をカンマ区切りで指定する。何も指定しない場合は、全て指定された状態になる。
- zoom：作成するベクトルタイルのズームレベルを指定する。なにも指定しない場合は、18となる。ただし、pathで指定したベクトルタイル（メッシュ単位）で生成していないズームレベルが指定された場合は出力されない。

### UPDATEモード
UPDATEモードは、すでにADD、MERGEによって「ベクトルタイル（メッシュ単位）」及び「ベクトルタイル」が生成されているとき、あるメッシュについて入力ファイルが更新された場合に実行することを想定している。  
UPDATEモードは、まずADDモードと同一のフローで新しい「ベクトルタイル（メッシュ単位）」を生成し、既に作成済の「ベクトルタイル（メッシュ単位）」との差分を確認し、更新があったタイルについて、「ベクトルタイル」の更新を実行する。  
UPDATEモードで指定できる引数及びオプションは次のとおり。
- mergedir：マージを区別する名称。MERGEモードで指定したmergedirと同じ値を指定する。
- base：出力するベクトルタイルのルートディレクトリを指定する。なにも指定しない場合は、カレントディレクトリが設定される。ADD, MERGEで作成した際と同じパスを指定する必要がある。
- feature：変換の対象とするデータ種別をカンマ区切りで指定する。何も指定しない場合は、全て指定された状態になる。
- zoom：作成するベクトルタイルのズームレベルを指定する。なにも指定しない場合は、18となる。
- s：入力ファイルがシェープファイルの場合は「-s」オプションを付ける。
