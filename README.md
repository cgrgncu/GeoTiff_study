# GeoTiff_study

### REF:
+ https://www.earthdatascience.org/courses/use-data-open-source-python/intro-raster-data-python/fundamentals-raster-data/intro-to-the-geotiff-file-format/
+ https://trac.osgeo.org/geotiff/
+ http://geotiff.maptools.org/spec/geotiff2.5.html
+ https://www.researchgate.net/publication/228077300_ETOPO1_1_Arc-Minute_Global_Relief_Model_procedures_data_sources_and_analysis


### 柵格資料(Raster Data)
柵格資料(Raster Data)又稱網格資料。在圖片的領域中常被應用於陣圖、像素圖。GeoTiff格式可以儲存此種資料。

### DTM、DEM、DSM
+ 數值地形模型（Digital Terrain Model，簡稱DTM）：以數值表示地形特徵，如高度、植被、建築物等等，並可計算坡度和坡向等資料。目前多採用網格模式的資訊系統來分析地形資料，且假設相鄰兩點間的地形是直線變化或圓滑變化。主流使用規則網格(regular grid)，長寬相等的網格連續分布。而DTM一般又可細分為DEM及DSM：
  + DEM：數值高程模型(Digltal Elevation Model)，不含地表植被、人工建物，僅有地面自然地形之起伏數值高度。
  + DSM：數值地表模型(Digital Surface Model)，包含地表植被、人工建物的地表起伏數值高度。
+ 最基本的描述方式是一個正方形區域有一個代表高程(通常是平均高程)，習慣性稱為Z值。可以儲存在XYZ格式中，其中X、Y是該正方形區域的中心點。因此要儲存一大塊區域切割成多個正方形所組成的區域，可以使用多個等間距的XYZ逐行寫下的方式儲存檔案。例如:

```
X[m], Y[m], Z[m]
0   , 0   , Z(0,0)=0
1   , 0   , Z(1,0)=0
2   , 0   , Z(2,0)=0
3   , 0   , Z(3,0)=0
4   , 0   , Z(4,0)=0
0   , 1   , Z(0,1)=0
1   , 1   , Z(1,1)=0
2   , 1   , Z(2,1)=5
3   , 1   , Z(3,1)=0
4   , 1   , Z(4,1)=0
0   , 2   , Z(0,2)=0
1   , 2   , Z(1,2)=0
2   , 2   , Z(2,2)=9
3   , 2   , Z(3,2)=0
4   , 2   , Z(4,2)=0
```
+ 繪圖:

```matlab
clear;clc;close all
%--
% 範例散點資料
xyz_data=[
0   , 0   , 0
1   , 0   , 0
2   , 0   , 0
3   , 0   , 0
4   , 0   , 0
0   , 1   , 0
1   , 1   , 0
2   , 1   , 5
3   , 1   , 0
4   , 1   , 0
0   , 2   , 0
1   , 2   , 0
2   , 2   , 9
3   , 2   , 0
4   , 2   , 0
0   , 3   , 0
1   , 3   , 0
2   , 3   , 0
3   , 3   , 0
4   , 3   , 0
];
%--
% 繪圖
figure
scatter(xyz_data(:,1),xyz_data(:,2),200,xyz_data(:,3),'filled')
hold on
for i=1:length(xyz_data(:,1))
	plot([-0.5,0.5,0.5,-0.5,-0.5]+xyz_data(i,1),[-0.5,-0.5,0.5,0.5,-0.5]+xyz_data(i,2),'r')
end
text(2   , 1   , '5[m]')
text(2   , 2   , '9[m]')
% 設定
title('網格中心點與網格實際涵蓋範圍')
grid on
box on
color_bar_handle=colorbar;
color_bar_title_handle=title(color_bar_handle,'Z[m]');
axis equal
xlabel('X[m]')
ylabel('Y[m]')
% 儲存檔案
saveas(gcf,'xyz_DTM_example','png')
%--
```

+ 等間距的方式則可以用二維陣列來儲存，也是一種圖片像素的方式儲存。
  + 原本XYZ需要儲存20x3=60個數字，現在只需要20x1=20個數字搭配起始點位置X0、起始點位置Y0、間隔距離delta_X、間隔距離delta_Y，共24個數字，大幅節省空間。
  + 例如，把左下當原點，X往右為正，Y往上為正，則:

	|Z(0,3)=0  |Z(1,3)=0  |Z(2,3)=0  |Z(3,3)=0  |Z(4,3)=0  |
	|----------|----------|----------|----------|----------|
	|Z(0,2)=0  |Z(1,2)=0  |Z(2,2)=9  |Z(3,2)=0  |Z(4,2)=0  |
	|Z(0,1)=0  |Z(1,1)=0  |Z(2,1)=5  |Z(3,1)=0  |Z(4,1)=0  |
	|Z(0,0)=0  |Z(1,0)=0  |Z(2,0)=0  |Z(3,0)=0  |Z(4,0)=0  |

+ 「grid-registered」與「cell-registered」
  + 依照美國國家海洋暨大氣總署(NOAA)的說明(ETOPO1_1_Arc-Minute_Global_Relief_Model_procedures.pdf的章節3.3.4)，單個網格會有一個平均高程值，如同前述的像素儲存。
    + 「grid-registered」模式下，將沿著目標網格範圍的實際線儲存其XYZ資料，會造成實際涵蓋的範圍超過目標網格範圍。例如，目標網格為台灣區域的經緯度(經度119 ~ 121[度]，緯度21 ~ 26[度])，間隔為1[度]，共計3x6=18組XYZ，如下:
	```
	X[m]  , Y[m] , Z[m]
	119   , 21   , Z(119,21)
	120   , 21   , Z(120,21)
	121   , 21   , Z(121,21)
	119   , 22   , Z(119,22)
	120   , 22   , Z(120,22)
	121   , 22   , Z(121,22)
	119   , 23   , Z(119,23)
	120   , 23   , Z(120,23)
	121   , 23   , Z(121,23)
	119   , 24   , Z(119,24)
	120   , 24   , Z(120,24)
	121   , 24   , Z(121,24)
	119   , 25   , Z(119,25)
	120   , 25   , Z(120,25)
	121   , 25   , Z(121,25)
	119   , 26   , Z(119,26)
	120   , 26   , Z(120,26)
	121   , 26   , Z(121,26)
	```    
    + 「cell-registered」模式下，將沿著目標網格範圍的實際線內縮一半間距來儲存其XYZ資料，使實際涵蓋的範圍等於目標網格範圍。例如，目標網格為台灣區域的經緯度(經度119 ~ 121[度]，緯度21 ~ 26[度])，間隔為1[度]，共計2x5=10組XYZ，如下:
	```
	X[m]  , Y[m] , Z[m]
	119.5 , 21.5 , Z(119.5,21.5)
	120.5 , 21.5 , Z(120.5,21.5)
	119.5 , 22.5 , Z(119.5,22.5)
	120.5 , 22.5 , Z(120.5,22.5)
	119.5 , 23.5 , Z(119.5,23.5)
	120.5 , 23.5 , Z(120.5,23.5)
	119.5 , 24.5 , Z(119.5,24.5)
	120.5 , 24.5 , Z(120.5,24.5)
	119.5 , 25.5 , Z(119.5,25.5)
	120.5 , 25.5 , Z(120.5,25.5)
	```
    + 這兩種模式主要與實際量測有關，習慣上實際量測會在經緯度整數值為中心點，自然會符合「grid-registered」模式，但有時考量儲存空間也有人使用「cell-registered」模式。比較麻煩的是這兩種模式的資料互相轉換時會依賴內插方法，因為某模式任意一個點對應另一模式中會有等距的4個點存在，故會造成有損轉換，以NOAA的作法，會告訴你原始的資料採用「grid-registered」，另外提供內插後的「cell-registered」資料。
### GeoTiff 
科學家採用圖片像素的方式去儲存DTM資料，其中一個有名的格式就是GeoTiff。GeoTiff格式符合知名標籤圖檔格式Tiff(Tagged Image File Format) 6.0。主要是在圖檔中嵌入地理資訊(包括地圖投影、坐標系、橢球體、基準面)，因此也享有Tiff檔案的優點(無損壓縮、靈活標籤定義)。

+ 在Tiff規範裡面，預設座標是X往右為正，Y往下為正。像素存放像是矩陣，用索引值(i,j)表示。
	|P(1,1)  |P(2,1)  |P(3,1)  |P(4,1)  |
	|--------|--------|--------|--------|
	|P(1,2)  |P(2,2)  |P(3,2)  |P(4,2)  |
	|P(1,3)  |P(2,3)  |P(3,3)  |P(4,3)  |
+ 另外會有許多GeoTiff規劃的標籤(Tag)，紀錄地理參數(包括地圖投影、坐標系、橢球體、基準面)以及二維陣列對應的原點、間隔。 

### ETOPO1
美國國家海洋暨大氣總署(NOAA)提供的「ETOPO1 Global Relief Model」是一個全球性的、「cell-registered」的、1弧分的網格，緯度從-90度到90度，經度從-180度到180度。
  + 下載頁面: https://www.ngdc.noaa.gov/mgg/global/
  + 其中可以下載「grid-registered」的版本與「cell-registered」版本。
  + 按照說明從檔案就可以知道尺寸有差異，「grid-registered」的版本是「寬=21601，高=10801」，「cell-registered」版本是「寬=21600，高=10800」。
  + 練習1:
    + 下載ETOPO1 Bedrock的grid-registered版本GeoTiff檔案(ETOPO1_Bed_g_geotiff.zip)，其檔案大小為312MB。解壓縮為Tiff檔案(ETOPO1_Bed_g_geotiff.tif)，其檔案大小為445MB。
    + 透過gdal_translate工具，將GeoTiff轉為XYZ。圖形化QGIS工具可以用，Raster>Conversion>Translate操作。檔名自訂，例如「output_g_xyz.xyz」，其檔案大小為9.79GB。
	 ```
	 gdal_translate -of XYZ C:/ETOPO1_Bed_g_geotiff/ETOPO1_Bed_g_geotiff.tif C:/ETOPO1_Bed_g_geotiff/output_g_xyz.xyz
	 ```
    + 讀取前5個點並顯示。
	```matlab
	clear;clc;close all
	%--
	% 讀檔案並顯示
	f1=fopen('output_g_xyz.xyz');
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	fclose(f1);
	%--
	% output_g_xyz.xyz前五行是:
	% -180 90 -4228
	% -179.98333333333332 90 -4228
	% -179.966666666666669 90 -4228
	% -179.949999999999989 90 -4228
	% -179.933333333333337 90 -4228
	```
  + 練習2:
    + 下載ETOPO1 Bedrock的cell-registered版本GeoTiff檔案(ETOPO1_Bed_c_geotiff.zip)，其檔案大小為312MB。解壓縮為Tiff檔案(ETOPO1_Bed_c_geotiff.tif)，其檔案大小為445MB。
    + 透過gdal_translate工具，將GeoTiff轉為XYZ。圖形化QGIS工具可以用，Raster>Conversion>Translate操作。檔名自訂，例如「output_c_xyz.xyz」，其檔案大小為9.80GB。
	 ```
	 gdal_translate -of XYZ C:/ETOPO1_Bed_c_geotiff/ETOPO1_Bed_c_geotiff.tif C:/ETOPO1_Bed_c_geotiff/output_c_xyz.xyz
	 ```
    + 讀取前5個點並顯示。
	```matlab
	clear;clc;close all
	%--
	% 讀檔案並顯示
	f1=fopen('output_c_xyz.xyz');
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	disp(fgetl(f1))
	fclose(f1);
	%--
	% output_g_xyz.xyz前五行是:
	% -179.991666666666674 89.99166666666666 -4229
	% -179.974999999999994 89.99166666666666 -4229
	% -179.958333333333343 89.99166666666666 -4229
	% -179.941666666666663 89.99166666666666 -4229
	% -179.925000000000011 89.99166666666666 -4229
	```
