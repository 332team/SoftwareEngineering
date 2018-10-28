```c++
# include "stdafx.h"

void setArea(int startY, int startX, int ylen, int xlen, int value, GByte* buff, int width) {
	for (int x = startX; x < startX + xlen; x++) {
		for (int y = startY; y < startY + ylen; y++) {
			buff[x * width + y] = value;
		}
	}
}

int main(){

	GDALDataset* src;
	GDALDataset* dst;

	char* srcPath = "src.jpg";
	char* dstPath = "dst.jpg";

	int width;
	int height;

	GByte* buff;
	GDALAllRegister();

	src = (GDALDataset*)GDALOpenShared(srcPath, GA_ReadOnly);
	width = src->GetRasterXSize();
	height = src->GetRasterYSize();

	buff = (GByte*) CPLMalloc(width * height * sizeof(GByte));

	int bandNum;
	bandNum = src->GetRasterCount();

	dst = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(
		dstPath, width, height, bandNum, GDT_Byte, NULL);

	for(int i = 0; i < bandNum; i++){
		src->GetRasterBand(i+1)->RasterIO(GF_Read, 0, 0, width, height
		, buff, width, height, GDT_Byte, 0, 0);

		if (i == 0) {
			setArea(100, 100, 200, 150, 255, buff, width);
		}

		setArea(300, 300, 100, 50, 255, buff, width);
		setArea(500, 500, 50, 100, 0, buff, width);

		dst->GetRasterBand(i+1)->RasterIO(GF_Write, 0, 0,
		width, height, buff, width, height, GDT_Byte, 0, 0);
	}


	CPLFree(buff);
	GDALClose(src);
	GDALClose(dst);
	return 0;
}

```

![](http://ww1.sinaimg.cn/large/0076RmPLly1fw7vl1mnbrj30s80kce81.jpg)