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
    + 讀取GeoTiff檔案並顯示部分資訊。
	```matlab
	clear;clc
	dem_info=imfinfo('ETOPO1_Bed_g_geotiff.tif');
	%--
	disp(dem_info)
	% 顯示如下:
	%                      Filename: 'ETOPO1_Bed_g_geotiff.tif'
	%                   FileModDate: '06-六月-2011 11:59:26'
	%                      FileSize: 466712272
	%                        Format: 'tif'
	%                 FormatVersion: []
	%                         Width: 21601
	%                        Height: 10801
	%                      BitDepth: 16
	%                     ColorType: 'grayscale'
	%               FormatSignature: [73 73 42 0]
	%                     ByteOrder: 'little-endian'
	%                NewSubFileType: 0
	%                 BitsPerSample: 16
	%                   Compression: 'Uncompressed'
	%     PhotometricInterpretation: 'BlackIsZero'
	%                  StripOffsets: [10801x1 double]
	%               SamplesPerPixel: 1
	%                  RowsPerStrip: 1
	%               StripByteCounts: [10801x1 double]
	%                   XResolution: []
	%                   YResolution: []
	%                ResolutionUnit: 'None'
	%                      Colormap: []
	%           PlanarConfiguration: 'Chunky'
	%                     TileWidth: []
	%                    TileLength: []
	%                   TileOffsets: []
	%                TileByteCounts: []
	%                   Orientation: 1
	%                     FillOrder: 1
	%              GrayResponseUnit: 0.0100
	%                MaxSampleValue: 65535
	%                MinSampleValue: 0
	%                  Thresholding: 1
	%                        Offset: 8
	%                  SampleFormat: 'Two's complement signed integer'
	%            ModelPixelScaleTag: [3x1 double]
	%              ModelTiepointTag: [6x1 double]
	%                 GDAL_METADATA: [1x783 char]
	%                   GDAL_NODATA: '-2147483648 '
	%--
	disp(dem_info.ModelPixelScaleTag)
	% 顯示如下:
	%     0.0167
	%     0.0167
	%          0
	%--
	disp(dem_info.ModelTiepointTag)
	% 顯示如下:
	%          0
	%          0
	%          0
	%  -180.0083
	%    90.0083
	%          0
	%--
	disp(dem_info.GDAL_METADATA)
	% 顯示如下:
	% <GDALMetadata>
	%   <Item name="NC_GLOBAL#Conventions">COARDS/CF-1.0</Item>
	%   <Item name="NC_GLOBAL#title">ETOPO1_Bed_g_gmt4.grd</Item>
	%   <Item name="NC_GLOBAL#history">grdreformat ETOPO1_Bed_g_gdal.grd ETOPO1_Bed_g_gmt4.grd=ni</Item>
	%   <Item name="NC_GLOBAL#GMT_version">4.4.0</Item>
	%   <Item name="NC_GLOBAL#node_offset">0</Item>
	%   <Item name="z#long_name">z</Item>
	%   <Item name="z#_FillValue">-2147483648</Item>
	%   <Item name="z#actual_range">-10898, 8271</Item>
	%   <Item name="x#long_name">Longitude</Item>
	%   <Item name="x#actual_range">-180, 180</Item>
	%   <Item name="x#units">degrees</Item>
	%   <Item name="y#long_name">Latitude</Item>
	%   <Item name="y#actual_range">-90, 90</Item>
	%   <Item name="y#units">degrees</Item>
	%   <Item name="NETCDF_VARNAME" sample="0">z</Item>
	% </GDALMetadata>
	%--
	dem_data=imread('ETOPO1_Bed_g_geotiff.tif');
	disp(dem_data(1:5,1:5))
	%--
	% 顯示如下:
	%   -4228  -4228  -4228  -4228  -4228
	%   -4229  -4229  -4229  -4229  -4229
	%   -4228  -4227  -4228  -4228  -4228
	%   -4225  -4225  -4225  -4225  -4225
	%   -4222  -4223  -4222  -4222  -4222
	%--
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
    + 讀取GeoTiff檔案並顯示部分資訊。
	```matlab
	clear;clc
	dem_info=imfinfo('ETOPO1_Bed_c_geotiff.tif');
	%--
	disp(dem_info)
	% 顯示如下:
	%                      Filename: 'ETOPO1_Bed_c_geotiff.tif'
	%                   FileModDate: '06-六月-2011 11:58:44'
	%                      FileSize: 466647468
	%                        Format: 'tif'
	%                 FormatVersion: []
	%                         Width: 21600
	%                        Height: 10800
	%                      BitDepth: 16
	%                     ColorType: 'grayscale'
	%               FormatSignature: [73 73 42 0]
	%                     ByteOrder: 'little-endian'
	%                NewSubFileType: 0
	%                 BitsPerSample: 16
	%                   Compression: 'Uncompressed'
	%     PhotometricInterpretation: 'BlackIsZero'
	%                  StripOffsets: [10800x1 double]
	%               SamplesPerPixel: 1
	%                  RowsPerStrip: 1
	%               StripByteCounts: [10800x1 double]
	%                   XResolution: []
	%                   YResolution: []
	%                ResolutionUnit: 'None'
	%                      Colormap: []
	%           PlanarConfiguration: 'Chunky'
	%                     TileWidth: []
	%                    TileLength: []
	%                   TileOffsets: []
	%                TileByteCounts: []
	%                   Orientation: 1
	%                     FillOrder: 1
	%              GrayResponseUnit: 0.0100
	%                MaxSampleValue: 65535
	%                MinSampleValue: 0
	%                  Thresholding: 1
	%                        Offset: 8
	%                  SampleFormat: 'Two's complement signed integer'
	%            ModelPixelScaleTag: [3x1 double]
	%              ModelTiepointTag: [6x1 double]
	%                 GDAL_METADATA: [1x789 char]
	%                   GDAL_NODATA: '-2147483648 '
	%--
	disp(dem_info.ModelPixelScaleTag)
	% 顯示如下:
	%     0.0167
	%     0.0167
	%          0
	%--
	disp(dem_info.ModelTiepointTag)
	% 顯示如下:
	%      0
	%      0
	%      0
	%   -180
	%     90
	%      0
	%--
	disp(dem_info.GDAL_METADATA)
	% 顯示如下:
	% <GDALMetadata>
	%   <Item name="NC_GLOBAL#Conventions">COARDS/CF-1.0</Item>
	%   <Item name="NC_GLOBAL#title">ETOPO1_Bed_c_gmt4.grd</Item>
	%   <Item name="NC_GLOBAL#history">grdsample -V ETOPO1_Bed_g_gmt4.grd -GETOPO1_Bed_c_gmt4.grd=ni -T</Item>
	%   <Item name="NC_GLOBAL#GMT_version">4.4.0</Item>
	%   <Item name="NC_GLOBAL#node_offset">1</Item>
	%   <Item name="z#long_name">z</Item>
	%   <Item name="z#_FillValue">-2147483648</Item>
	%   <Item name="z#actual_range">-10803, 8333</Item>
	%   <Item name="x#long_name">Longitude</Item>
	%   <Item name="x#actual_range">-180, 180</Item>
	%   <Item name="x#units">degrees</Item>
	%   <Item name="y#long_name">Latitude</Item>
	%   <Item name="y#actual_range">-90, 90</Item>
	%   <Item name="y#units">degrees</Item>
	%   <Item name="NETCDF_VARNAME" sample="0">z</Item>
	% </GDALMetadata>
	%--
	dem_data=imread('ETOPO1_Bed_c_geotiff.tif');
	disp(dem_data(1:5,1:5))
	%--
	% 顯示如下:
	%   -4229  -4229  -4229  -4229  -4229
	%   -4228  -4228  -4229  -4229  -4228
	%   -4226  -4226  -4227  -4227  -4226
	%   -4224  -4224  -4223  -4223  -4224
	%   -4224  -4223  -4223  -4223  -4223
	%--
	```
	> 至此，可以確認前述兩個模式，以及在Tiff格式中儲存的資訊差異。另外，對照XYZ檔案與從GeoTiff取出的二維陣列資料，應該可以寫出正確的轉換程式。從Tiff標籤資訊及二維陣列可知，ETOPO的高程解析度是1[m]，使用16位元整數值(-32768 ~ 32767)儲存。剩餘資訊可以慢慢看，但因為各家格式不同，會記錄的標籤也不同，目前沒有興趣理解各家軟體會輸出怎樣的Tiff，但大原則應該是相同的。
  + 練習3:
    + 下載ETOPO1 Bedrock的grid-registered版本GeoTiff檔案(ETOPO1_Bed_g_geotiff.zip)，其檔案大小為312MB。解壓縮為Tiff檔案(ETOPO1_Bed_g_geotiff.tif)，其檔案大小為445MB。
    + 透過gdal_translate工具，將GeoTiff轉為XYZ。圖形化QGIS工具可以用，Raster>Conversion>Translate操作。檔名自訂，例如「output_g_xyz.xyz」，其檔案大小為9.79GB。
	 ```
	 gdal_translate -of XYZ C:/ETOPO1_Bed_g_geotiff/ETOPO1_Bed_g_geotiff.tif C:/ETOPO1_Bed_g_geotiff/output_g_xyz.xyz
	 ```
    + 把xyz檔案轉檔為mat檔，避免重複讀檔耗費太多時間。以目前的電腦讀一次xyz(9.79GB)可能要20分鐘。
	```matlab
	clear;clc
	%--
	% 轉檔(xyz to mat)
	% 因原本xyz檔案太大，重複讀取太耗時間，故將資料改存為mat檔。
	%--
	tic
	dem_xyz=load('111.xyz');
	toc
	% <233312401x3 double>
	% Elapsed time is 1164.683978 seconds.
	tic
	save('dem_xyz','dem_xyz','-v7.3');
	toc
	% Elapsed time is 52.318456 seconds.
	%--
	```
    + 比較xyz與GeoTiff差異。
	```matlab
	clear;clc
	%--
	% 比較xyz與GeoTiff差異
	%--
	% 讀GeoTiff檔
	tic
	dem_geotiff_data=imread('ETOPO1_Bed_g_geotiff.tif');
	toc
	% <10801x21601 int16>
	% Elapsed time is 3.071906 seconds.
	%--
	tic
	%--
	dem_geotiff_info=imfinfo('ETOPO1_Bed_g_geotiff.tif');
	toc
	% Elapsed time is 0.002642 seconds.
	%--
	%
	disp(dem_geotiff_info.Width)
	%        21601
	disp(dem_geotiff_info.Height)
	%        10801
	%--
	% ModelPixelScaleTag (ScaleX, ScaleY, ScaleZ)
	% ScaleX : 水平方向間距(單位要另外查詢)
	% ScaleY : 垂直方向間距(單位要另外查詢)
	% ScaleZ : 一般多屬於地圖，為2D狀況，此項填0，
	disp(dem_geotiff_info.ModelPixelScaleTag)
	%     0.0167
	%     0.0167
	%          0
	%--
	% ModelTiepointTag
	% 位置(I,J,K)的像素點對應實際(X,Y,Z)。
	% 一般多屬於地圖，為2D狀況，K、Z填0。
	disp(dem_geotiff_info.ModelTiepointTag)
	%          0
	%          0
	%          0
	%  -180.0083
	%    90.0083
	%          0
	% 以上描述原點(0,0,0)的像素對應實際座標(-180.0083,90.0083,0)。
	% 請注意數值位數顯示只有部分，實際上後面還有很多位數。
	%--
	X_vector=(0:dem_geotiff_info.Width-1)*dem_geotiff_info.ModelPixelScaleTag(1)+dem_geotiff_info.ModelTiepointTag(4)+(dem_geotiff_info.ModelPixelScaleTag(1)/2);
	disp('X_vector:')
	disp(num2str(X_vector(1:5)))
	% -180     -179.9833     -179.9667       -179.95     -179.9333
	Y_vector=(0:-1:-(dem_geotiff_info.Height-1))*dem_geotiff_info.ModelPixelScaleTag(2)+dem_geotiff_info.ModelTiepointTag(5)-(dem_geotiff_info.ModelPixelScaleTag(2)/2);
	disp('Y_vector:')
	disp(num2str(Y_vector(1:5)))
	% 90      89.9833      89.9667        89.95      89.9333
	%--
	% 讀xyz檔
	tic
	load('dem_xyz.mat')
	toc
	% <233312401x3 double>
	% Elapsed time is 13.343948 seconds.
	%--
	% 比較
	disp('GeoTiff 2D Array:')
	disp(dem_geotiff_data(1:5,1:5))
	% GeoTiff 2D Array:
	%   -4228  -4228  -4228  -4228  -4228
	%   -4229  -4229  -4229  -4229  -4229
	%   -4228  -4227  -4228  -4228  -4228
	%   -4225  -4225  -4225  -4225  -4225
	%   -4222  -4223  -4222  -4222  -4222
	disp('XYZ Data:')
	disp(num2str(dem_xyz([1:5],1:3)));
	disp(num2str(dem_xyz([1:5]+21601,1:3)));
	disp(num2str(dem_xyz([1:5]+21601*2,1:3)));
	disp(num2str(dem_xyz([1:5]+21601*3,1:3)));
	disp(num2str(dem_xyz([1:5]+21601*4,1:3)));
	% XYZ Data:
	%       -180             90          -4228
	% -179.98333             90          -4228
	% -179.96667             90          -4228
	%    -179.95             90          -4228
	% -179.93333             90          -4228
	%       -180      89.983333          -4229
	% -179.98333      89.983333          -4229
	% -179.96667      89.983333          -4229
	%    -179.95      89.983333          -4229
	% -179.93333      89.983333          -4229
	%       -180      89.966667          -4228
	% -179.98333      89.966667          -4227
	% -179.96667      89.966667          -4228
	%    -179.95      89.966667          -4228
	% -179.93333      89.966667          -4228
	%       -180          89.95          -4225
	% -179.98333          89.95          -4225
	% -179.96667          89.95          -4225
	%    -179.95          89.95          -4225
	% -179.93333          89.95          -4225
	%       -180      89.933333          -4222
	% -179.98333      89.933333          -4223
	% -179.96667      89.933333          -4222
	%    -179.95      89.933333          -4222
	% -179.93333      89.933333          -4222
	%--
	```
	> 證實Z的部分存放在像素中，X、Y可利用等間距算出來。

### 用經緯度地圖來解釋Grid註冊模式的GeoTiff存放方式
+ 讀取前5個點並顯示。
	```matlab
	clear;clc;close all
	%--------------------------------------------------------------------------
	% DEM的xyz格式資料排序說明:
	%--------------------------------------------------------------------------
	% 製作範例資料(普通照順序的想法，不是DEM習慣的格式)
	[xi,yi] = meshgrid(1:5,1:4);
	zi=zeros(size(xi));
	% 填入指定數字
	for i=1:numel(zi)
	    zi(i)=i;
	end
	%--------------------------------------------------------------------------
	% 解析:
	% xi,yi,zi尺寸都是mxn。本範例中:
	% xi =
	% 
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	% 
	% yi =
	% 
	%      1     1     1     1     1
	%      2     2     2     2     2
	%      3     3     3     3     3
	%      4     4     4     4     4
	% 
	% zi =
	% 
	%      0     0     0     0     0
	%      0     0     0     0     0
	%      0     0     0     0     0
	%      0     0     0     0     0
	%--------------------------------------------------------------------------
	% 轉成散點資料
	all_node_list=[xi(:) yi(:) zi(:)];
	% all_node_list =
	%      1     1     1
	%      1     2     2
	%      1     3     3
	%      1     4     4
	%      2     1     5
	%      2     2     6
	%      2     3     7
	%      2     4     8
	%      3     1     9
	%      3     2    10
	%      3     3    11
	%      3     4    12
	%      4     1    13
	%      4     2    14
	%      4     3    15
	%      4     4    16
	%      5     1    17
	%      5     2    18
	%      5     3    19
	%      5     4    20
	%--------------------------------------------------------------------------
	% 科技部提供的標準DEM的xyz資料會是如下排序
	%--
	% 重新排序
	dem_node_list=sortrows(all_node_list,[-2,1,3]);
	% dem_node_list =
	% 
	%      1     4     4
	%      2     4     8
	%      3     4    12
	%      4     4    16
	%      5     4    20
	%      1     3     3
	%      2     3     7
	%      3     3    11
	%      4     3    15
	%      5     3    19
	%      1     2     2
	%      2     2     6
	%      3     2    10
	%      4     2    14
	%      5     2    18
	%      1     1     1
	%      2     1     5
	%      3     1     9
	%      4     1    13
	%      5     1    17
	%--
	% 因為sortrows效率很差，所以每次都轉回習慣的xi往右為正，yi往下為正的話會耗費很多時間
	% 所以可以直接把標準DEM的XYZ資料依照其排序放入dem_xi、dem_yi、dem_zi
	X_Tick_count=sum(dem_node_list(:,2)==dem_node_list(1,2));
	Y_Tick_count=sum(dem_node_list(:,1)==dem_node_list(1,1));
	if ((X_Tick_count*Y_Tick_count)==length(dem_node_list(:,1)))
	    disp('數量正確!')
	else
	    disp('數量錯誤!非DEM的XYZ資料')
	    return
	end
	dem_xi=reshape(dem_node_list(:,1),X_Tick_count,[])';
	dem_yi=reshape(dem_node_list(:,2),X_Tick_count,[])';
	dem_zi=reshape(dem_node_list(:,3),X_Tick_count,[])';
	%--------------------------------------------------------------------------
	% 解析:
	% xi,yi,zi尺寸都是mxn。本範例中:
	% dem_xi =
	% 
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	% 
	% dem_yi =
	% 
	%      4     4     4     4     4
	%      3     3     3     3     3
	%      2     2     2     2     2
	%      1     1     1     1     1
	% 
	% dem_zi =
	% 
	%      4     8    12    16    20
	%      3     7    11    15    19
	%      2     6    10    14    18
	%      1     5     9    13    17
	%--------------------------------------------------------------------------
	% 其中 dem_zi 是 < 4x5 double >
	% 寬是5個元素，高是4個元素
	% dem_zi內容就是GeoTiff陣列的儲存方式，dem_xi、dem_yi可以靠其他資訊算出來:
	dem_x_tick_vector=1:5;
	dem_y_tick_vector=4:-1:1;
	[my_dem_xi,my_dem_yi]=meshgrid(dem_x_tick_vector,dem_y_tick_vector);
	% my_dem_xi =
	% 
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	%      1     2     3     4     5
	% 
	% my_dem_yi =
	% 
	%      4     4     4     4     4
	%      3     3     3     3     3
	%      2     2     2     2     2
	%      1     1     1     1     1
	% 
	%--------------------------------------------------------------------------
	```

### 內政部DEM  
+ REF: https://www.tgos.tw/TGOS/Web/TGOS_Home.aspx  
+ 2016年版本  
  + 全臺灣20公尺網格間距的數值地形模型（DTM）資料，每一個網格點記錄該點之平面坐標與高程資料  
  + 圖資釋出時間: 2016-08-29  
  + 座標系統: EPSG3826(TWD97121分帶)  
  + 來源: https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612640  
  + 下載連結: http://dtm.moi.gov.tw/不分幅_全台及澎湖.zip  
    + 檔案大小: 101 MB (106,162,957 位元組)  
+ 2018年版本(彙整2016年、2017年計畫成果後發布)  
  + 2018年產製之臺灣（本島除樂山管制區外）20公尺網格間距的數值地形模型（DTM）資料，每一個網格點記錄該點之平面坐標與高程資料  
  + 圖資釋出時間: 2018-07-27  
  + 座標系統: EPSG3826(TWD97121分帶)  
  + 來源: https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612686  
  + 下載連結: https://dtm.moi.gov.tw/全台灣新版_不分幅_臺灣本島.zip  
    + 檔案大小: 299 MB (314,426,160 位元組)  
+ 2019年版本  
  + 2019新版之臺灣（本島除樂山管制區外）20公尺網格間距的數值地形模型（DTM）資料，每一個網格點記錄該點之平面坐標與高程資料  
  + 圖資釋出時間: 2019-06-17  
  + 座標系統: EPSG3826(TWD97121分帶)  
  + 來源: https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612704 
  + 下載連結: https://dtm.moi.gov.tw/tif/taiwan_TIF格式.7z  
    + 檔案大小: 176 MB (184,670,886 位元組)    
+ 2020年版本  
  + 圖資釋出時間:2020-11-17  
  + 座標系統:EPSG3826(TWD97121分帶)  
  + 來源: https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612706
  + 下載連結: https://dtm.moi.gov.tw/2020dtm20m/台灣本島及4離島(龜山島_綠島_蘭嶼_小琉球).7z
    + 檔案大小: 176 MB (184,776,118 位元組)  
+ 2021年版本
+ 2022年版本
