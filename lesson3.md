# lesson3 

```c++
# include "stdafx.h"

int main(){

	GDALDataset* bg;
	GDALDataset* fg;
	GDALDataset* dst;

	char* bgPath = "space.jpg";
	char* fgPath = "superman.jpg";
	char* dstPath  ="dst.jpg";

	int width;
	int height;

	GByte* buff_bg_r;
	GByte* buff_bg_g;
	GByte* buff_bg_b;
	GByte* buff_fg_r;
	GByte* buff_fg_g;
	GByte* buff_fg_b;
	GDALAllRegister();

	bg = (GDALDataset*)GDALOpenShared(bgPath, GA_ReadOnly);
	fg = (GDALDataset*)GDALOpenShared(fgPath, GA_ReadOnly);
	width = bg->GetRasterXSize();
	height = bg->GetRasterYSize();

	buff_bg_r = (GByte*) CPLMalloc(width * height * sizeof(GByte));
	buff_bg_g = (GByte*) CPLMalloc(width * height * sizeof(GByte));
	buff_bg_b = (GByte*) CPLMalloc(width * height * sizeof(GByte));
	buff_fg_r = (GByte*) CPLMalloc(width * height * sizeof(GByte));
	buff_fg_g = (GByte*) CPLMalloc(width * height * sizeof(GByte));
	buff_fg_b = (GByte*) CPLMalloc(width * height * sizeof(GByte));

	int bandNum;
	bandNum = bg->GetRasterCount();

	dst = GetGDALDriverManager()->GetDriverByName("GTiff")->Create(
		dstPath, width, height, bandNum, GDT_Byte, NULL);

	bg->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, width, height, buff_bg_r, width, height, GDT_Byte, 0, 0);
	bg->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, width, height, buff_bg_g, width, height, GDT_Byte, 0, 0);
	bg->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, width, height, buff_bg_b, width, height, GDT_Byte, 0, 0);

	fg->GetRasterBand(1)->RasterIO(GF_Read, 0, 0, width, height, buff_fg_r, width, height, GDT_Byte, 0, 0);
	fg->GetRasterBand(2)->RasterIO(GF_Read, 0, 0, width, height, buff_fg_g, width, height, GDT_Byte, 0, 0);
	fg->GetRasterBand(3)->RasterIO(GF_Read, 0, 0, width, height, buff_fg_b, width, height, GDT_Byte, 0, 0);

	for (int i = 0; i < height; i++) {
		for (int j = 0; j < width; j++) {
			int index = i * width + j;
			bool shouldReplace = false;
			if ((10 >= buff_fg_r[index]) || (buff_fg_r[index] >= 160)) shouldReplace = true;
			if ((100 >= buff_fg_g[index]) || (buff_fg_g[index] >= 220)) shouldReplace = true;
			if ((10 >= buff_fg_b[index]) || (buff_fg_b[index] >= 110)) shouldReplace = true;
			if (shouldReplace) {
				buff_bg_r[index] = buff_fg_r[index];
				buff_bg_g[index] = buff_fg_g[index];
				buff_bg_b[index] = buff_fg_b[index];
			}
		}
	}

	dst->GetRasterBand(1)->RasterIO(GF_Write, 0, 0,
		width, height, buff_bg_r, width, height, GDT_Byte, 0, 0);
	dst->GetRasterBand(2)->RasterIO(GF_Write, 0, 0,
		width, height, buff_bg_g, width, height, GDT_Byte, 0, 0);
	dst->GetRasterBand(3)->RasterIO(GF_Write, 0, 0,
		width, height, buff_bg_b, width, height, GDT_Byte, 0, 0);

	CPLFree(buff_bg_r);
	CPLFree(buff_bg_g);
	CPLFree(buff_bg_b);
	CPLFree(buff_fg_r);
	CPLFree(buff_fg_g);
	CPLFree(buff_fg_b);
	GDALClose(bg);
	GDALClose(fg);
	GDALClose(dst);
	return 0;
}

```

![](http://ww1.sinaimg.cn/large/0076RmPLly1fwo540tlj5j30hs0dc7d8.jpg)