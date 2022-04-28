# GeoTiff_study

### REF:
+ https://www.earthdatascience.org/courses/use-data-open-source-python/intro-raster-data-python/fundamentals-raster-data/intro-to-the-geotiff-file-format/
+ https://trac.osgeo.org/geotiff/
+ http://geotiff.maptools.org/spec/geotiff2.5.html


### 柵格資料(Raster Data)
柵格資料(Raster Data)又稱網格資料。在圖片的領域中常被應用於陣圖、像素圖。GeoTiff也是屬於此種資料。
+ 像素是點還是面?
  + 在Tiff規範裡面，預設座標是X往右為正，Y往下為正。像素存放像是矩陣，用索引值(i,j)表示。

|P(1,1)  |P(2,1)  |P(3,1)  |P(4,1)  |
|--------|--------|--------|--------|
|P(1,2)  |P(2,2)  |P(3,2)  |P(4,2)  |
|P(1,3)  |P(2,3)  |P(3,3)  |P(4,3)  |

  + 依照「PixelIsArea」(像素是面)，也有文獻稱為「grid-registered」。假設網格X方向與Y方向的間距為20公尺，則第一個像素P(1,1)所代表的是(0[m],0[m])、(20[m],0[m])、(20[m],20[m])、(0[m],20[m])四個點所圍住的400平方公尺的區域。第二個像素P(2,1)所代表的是(20[m],0[m])、(40[m],0[m])、(40[m],20[m])、(20[m],20[m])四個點所圍住的400平方公尺的區域。如前圖所示，有P(1,1)、P(2,1)、P(3,1)、P(4,1)、P(1,2)、P(2,2)、P(3,2)、P(4,2)、P(1,3)、P(2,3)、P(3,3)、P(4,3)共12個像素會被認為提供了(0[m],0[m])、(80[m],0[m])、(80[m],60[m])、(0[m],60[m])四個點所圍住的4800平方公尺的區域的資料。
  + 依照「PixelIsPoint」(像素是點)，也有文獻稱為「cell-registered」。假設網格X方向與Y方向的間距為20公尺，則第一個像素P(1,1)所代表的是(-10[m],-10[m])、(10[m],-10[m])、(10[m],10[m])、(-10[m],10[m])四個點所圍住的400平方公尺的區域。第二個像素P(2,1)所代表的是(10[m],-10[m])、(30[m],10[m])、(40[m],20[m])、(20[m],20[m])四個點所圍住的400平方公尺的區域。如前圖所示，有P(1,1)、P(2,1)、P(3,1)、P(4,1)、P(1,2)、P(2,2)、P(3,2)、P(4,2)、P(1,3)、P(2,3)、P(3,3)、P(4,3)共12個像素會被認為提供了(0[m],0[m])、(60[m],0[m])、(60[m],40[m])、(0[m],40[m])四個點所圍住的2400平方公尺的區域的資料。

### ETOPO1
ETOPO1 Global Relief Model 是一個全球性的、「cell-registered」的、1弧分的網格，緯度從-90度到90度，經度從-180度到180度。
  + 下載頁面: https://www.ngdc.noaa.gov/mgg/global/
  + 其中可以下載「grid-registered」的版本與「cell-registered」版本。
