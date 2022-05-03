# GeoTiff_study

### REF:
+ https://www.earthdatascience.org/courses/use-data-open-source-python/intro-raster-data-python/fundamentals-raster-data/intro-to-the-geotiff-file-format/
+ https://trac.osgeo.org/geotiff/
+ http://geotiff.maptools.org/spec/geotiff2.5.html


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
> 繪圖:

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
	+

### GeoTiff 
科學家採用圖片像素的方式去儲存DTM資料，其中一個有名的格式就是GeoTiff。GeoTiff格式符合知名標籤圖檔格式Tiff(Tagged Image File Format) 6.0。主要是在圖檔中嵌入地理資訊(包括地圖投影、坐標系、橢球體、基準面)，因此也享有Tiff檔案的優點(無損壓縮、靈活標籤定義)。

+ 在Tiff規範裡面，預設座標是X往右為正，Y往下為正。像素存放像是矩陣，用索引值(i,j)表示。

|P(1,1)  |P(2,1)  |P(3,1)  |P(4,1)  |
|--------|--------|--------|--------|
|P(1,2)  |P(2,2)  |P(3,2)  |P(4,2)  |
|P(1,3)  |P(2,3)  |P(3,3)  |P(4,3)  |


+ 「grid-registered」與「cell-registered」。
  + 下載頁面: https://www.ngdc.noaa.gov/mgg/global/
+ 「grid-registered」與「cell-registered」
+ 
+ 
+ 
+ + 
  + 依照「PixelIsArea」(像素是面)，也有文獻稱為「grid-registered」。假設網格X方向與Y方向的間距為20公尺，則第一個像素P(1,1)所代表的是(0[m],0[m])、(20[m],0[m])、(20[m],20[m])、(0[m],20[m])四個點所圍住的400平方公尺的區域。第二個像素P(2,1)所代表的是(20[m],0[m])、(40[m],0[m])、(40[m],20[m])、(20[m],20[m])四個點所圍住的400平方公尺的區域。如前圖所示，有P(1,1)、P(2,1)、P(3,1)、P(4,1)、P(1,2)、P(2,2)、P(3,2)、P(4,2)、P(1,3)、P(2,3)、P(3,3)、P(4,3)共12個像素會被認為提供了(0[m],0[m])、(80[m],0[m])、(80[m],60[m])、(0[m],60[m])四個點所圍住的4800平方公尺的區域的資料。
  + 依照「PixelIsPoint」(像素是點)，也有文獻稱為「cell-registered」。假設網格X方向與Y方向的間距為20公尺，則第一個像素P(1,1)所代表的是(-10[m],-10[m])、(10[m],-10[m])、(10[m],10[m])、(-10[m],10[m])四個點所圍住的400平方公尺的區域。第二個像素P(2,1)所代表的是(10[m],-10[m])、(30[m],10[m])、(40[m],20[m])、(20[m],20[m])四個點所圍住的400平方公尺的區域。如前圖所示，有P(1,1)、P(2,1)、P(3,1)、P(4,1)、P(1,2)、P(2,2)、P(3,2)、P(4,2)、P(1,3)、P(2,3)、P(3,3)、P(4,3)共12個像素會被認為提供了(0[m],0[m])、(60[m],0[m])、(60[m],40[m])、(0[m],40[m])四個點所圍住的2400平方公尺的區域的資料。

### ETOPO1
ETOPO1 Global Relief Model 是一個全球性的、「cell-registered」的、1弧分的網格，緯度從-90度到90度，經度從-180度到180度。
  + 下載頁面: https://www.ngdc.noaa.gov/mgg/global/
  + 其中可以下載「grid-registered」的版本與「cell-registered」版本。
  + 按照說明從檔案就可以知道尺寸有差異，「grid-registered」的版本是「寬=21601，高=10801」，「cell-registered」版本是「寬=21600，高=10800」。
