---
title: 師大資工程式設計二 期中考解題心得
date: 2024-04-15 23:00:00.000000000 +0800 CST
tags: [NTNUCSIE, Computer Programming II, Mid]
categories: [NTNUCSIE, Computer Programming II]
math: true
---


## 前言

[題目](https://drive.google.com/file/d/1HhX97mzHtCqlJ0qMEVxLFfzy-yT4CT5M/view)
<!-- more -->
這次整體難度比上學期期中考跟期末考難度低，加上老師跟助教有提示要先去寫處理 BMP file 的 function，所以寫得很順。

## 第一題 Matrix Multiplication

第一題是要處理矩陣運算，但不會給你矩陣的大小，所以要先把整個矩陣用 `fgets` 讀進來再處理，雖然我這題早就有寫好的 vector header 可以用來存矩陣，但因為處理輸入太花時間，所以跳過。

得分：**0 / 25**

## 第二題 Text Compression

第二題是要處理字串壓縮，這題我把字元對應的二進製讀進來，並用 `strtol` 來轉成十進位再丟到開好大小為 $29$ 的 array 裡，然後同時儲存長度來處理開頭是 $0$ 的情況。接下來就是開一個 byte 暫存然後計算暫存長度跟對應到字元的長度是否 $\geq 8$ ，如果是就把暫存的 byte 寫入檔案，然後把暫存的 byte 清空，直到該字元加字串的長度$\lt 8$，中間合併字串就只是單純的位元運算，然後等把字串全部寫完後，再把處理最後的 padding。

得分：**25 / 25**

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

int main()
{
    char file_name[5000];
    printf("Codebook: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *codebook = fopen(file_name, "r");
    if (codebook == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    printf("Input File: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *input = fopen(file_name, "rb");
    if (input == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    printf("Output File: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *output = fopen(file_name, "wb");
    if (output == NULL)
    {
        printf("No file.\n");
        return -1;
    }

    int32_t encode[29];
    int32_t encode_len[29] = {0};
    for (int32_t i = 0; i < 29; i++)
    {
        encode[i] = -1;
    }
    char code[100];
    while (!feof(codebook))
    {
        fgets(code, 100, codebook);
        if (code[0] == '\n')
        {
            continue;
            ;
        }
        if (code[1] == ':')
        {
            if (code[0] >= 'a' && code[0] <= 'z')
            {
                encode[code[0] - 'a'] = strtol(code + 3, NULL, 2);
                encode_len[code[0] - 'a'] = strlen(code + 3) - 1;
            }
            else
            {
                printf("Invalid codebook.\n");
                goto terminate;
            }
        }
        else
        {
            if (strstr(code, "space:"))
            {
                encode[26] = strtol(strchr(code, ':') + 2, NULL, 2);
                encode_len[26] = strlen(strchr(code, ':') + 2) - 1;
            }
            else if (strstr(code, "comma:"))
            {
                encode[27] = strtol(strchr(code, ':') + 2, NULL, 2);
                encode_len[27] = strlen(strchr(code, ':') + 2) - 1;
            }
            else if (strstr(code, "period:"))
            {
                encode[28] = strtol(strchr(code, ':') + 2, NULL, 2);
                encode_len[28] = strlen(strchr(code, ':') + 2) - 1;
            }
            else
            {
                printf("Invalid codebook.\n");
                goto terminate;
            }
        }
    }
    fclose(codebook);
    int8_t temp = 0;
    int8_t temp_len = 0;
    while (!feof(input))
    {

        char c = fgetc(input);
        if (c == EOF)
        {
            break;
        }
        int32_t code_now = 0;
        int32_t code_now_len = 0;
        if (c >= 'a' && c <= 'z')
        {
            code_now = encode[c - 'a'];
            code_now_len = encode_len[c - 'a'];
        }
        else if (c == ' ')
        {
            code_now = encode[26];
            code_now_len = encode_len[26];
        }
        else if (c == ',')
        {
            code_now = encode[27];
            code_now_len = encode_len[27];
        }
        else if (c == '.')
        {
            code_now = encode[28];
            code_now_len = encode_len[28];
        }
        else
        {
            printf("Invalid input.\n");
            goto terminate;
        }
        if (code_now == -1)
        {
            printf("Invalid codebook.\n");
            goto terminate;
        }
        // fprintf(stderr, "*%d %d\n", code_now, code_now_len);
        while (temp_len + code_now_len > 8)
        {
            temp = (temp << (8 - temp_len)) | (code_now >> (code_now_len - (8 - temp_len)));
            // fprintf(stderr, "%d ", temp);
            fwrite(&temp, 1, 1, output);
            code_now = code_now & ((1 << (code_now_len - (8 - temp_len))) - 1);
            code_now_len = code_now_len - (8 - temp_len);
            temp = 0;
            temp_len = 0;
        }
        temp = temp << code_now_len | code_now;
        temp_len += code_now_len;
        // fprintf(stderr, "(%d %d\n", temp, temp_len);
        if (temp_len == 8)
        {
            // fprintf(stderr, "a%d ", temp);
            fwrite(&temp, 1, 1, output);
            temp = 0;
            temp_len = 0;
        }
    }

    if (temp_len != 0)
    {
        temp = temp << (8 - temp_len);
        temp += 1 << (8 - temp_len - 1);
        fwrite(&temp, 1, 1, output);
    }
    else
    {
        temp = 0b10000000;
        fwrite(&temp, 1, 1, output);
    }
    fclose(input);
    fclose(output);
    return 0;
terminate:
    fclose(codebook);
    fclose(input);
    fclose(output);
    return -1;
}
```

## 第三題 Face/Off

這題因為我之前沒準備到縮放的 function，所以是寫完第二題才回來寫，這題因為我之前準備的 header file 裡的 function 是寫讀指定位置那個 pixel 的值，用 `fseek` 跳來跳去，所以很方便，然後在考試時就趕工寫縮放的 function，因為說是線性縮放，我不確定助教意思是什麼，所以我就直接算比例然後把對應的位置強制轉型成 `int32_t` 然後去讀取。
當我把 header file 的 function 寫完後，這題就只要 `for` 迴圈跑過整個圖片，把 y 值上下反轉然後再對應的範圍內寫入新的圖片就好。

得分：**5 / 25**

我居然忘記把高度再往下偏移，也就是抓到的點是左下不是左上，最後只拿到錯誤處理的 5 分。

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include "mybmp.h"

int main()
{
    char file_name[5000];
    printf("cover: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *cover = fopen(file_name, "rb");
    if (cover == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    printf("x (in pixel): ");
    int32_t x;
    scanf("%d", &x);
    printf("y (in pixel): ");
    int32_t y;
    scanf("%d", &y);
    printf("w (in pixel): ");
    int32_t w;
    scanf("%d", &w);
    printf("h (in pixel): ");
    int32_t h;
    scanf("%d", &h);
    printf("new: ");
    fgetc(stdin);
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *new = fopen(file_name, "rb");
    if (new == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    printf("output: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *output = fopen(file_name, "wb");
    if (output == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    sBmpHeader cover_header = read_header(cover);
    FILE *image_write = fopen("tmp.bmp", "wb");
    scale_image(new, image_write, w, h);
    fclose(image_write);
    fclose(new);
    new = fopen("tmp.bmp", "rb");
    sBmpHeader new_header = read_header(new);
    fwrite(&cover_header, sizeof(sBmpHeader), 1, output);
    // forgot in mid
    y+=new_header.height;
    // forgot in mid
    // y = cover_header.width - y;
    for (int32_t i = 0; i < abs(cover_header.height); i++)
    {
        for (int32_t j = 0; j < cover_header.width; j++)
        {
            sBmpPixel24 pixel;
            if (i >= cover_header.height - y && i < cover_header.height - y + h && j >= x && j < x + w)
            {
                pixel = read_pixel(new, new_header, i - (cover_header.height - y), j - x, 0);
            }
            else
            {
                pixel = read_pixel(cover, cover_header, i, j, 0);
            }
            fwrite(&pixel, sizeof(sBmpPixel24), 1, output);
        }
        write_edge_pixel(cover_header.width, output);
    }
    fclose(cover);
    fclose(new);
    fclose(output);
    remove("tmp.bmp");

    return 0;
}
```

## 第四題 Colorful Fantastic Render System []

這題是最簡單的，只要先設一個 $256 \times 256$ 的 array 來存顏色，設定顏色跟方向循環用的 array，存現在位置，再把每一個指令射程 function 來處理就好，然後中括號的循環就是用遞迴來處理，就是從 '\[' 讀到 '\]' 然後跑兩次就好。

得分：**23 / 25**
我忘了把顏色填入陣列了，兩分顏色處理就沒了。

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include "mybmp.h"

#define WIDTH 256
#define HEIGHT 256
int32_t di[8] = {1, 1, 0, -1, -1, -1, 0, 1};
int32_t dj[8] = {0, 1, 1, 1, 0, -1, -1, -1};
int32_t ri = 0, rj = 0;
int32_t now_x = 127, now_y = 127;
int32_t color_now = 0;
sBmpPixel24 color_draw[8];
sBmpPixel24 picture[256][256];

int8_t F_command()
{
    now_x += di[ri];
    now_y += dj[rj];
    if (now_x < 0 || now_x >= WIDTH || now_y < 0 || now_y >= HEIGHT)
    {
        return -1;
    }
    picture[now_x][now_y] = color_draw[color_now];
    return 0;
}
int8_t R_command()
{
    ri = (ri + 1) % 8;
    rj = (rj + 1) % 8;
    return 0;
}

int8_t C_command()
{
    color_now = (color_now + 1) % 8;
    return 0;
}

int8_t loop_command(int32_t *code_n, char *code)
{
    (*code_n)++;
    int32_t temp = *code_n;
    while (code[*code_n] != ']')
    {
        if (code[*code_n] == '[')
        {
            loop_command(code_n, code);
        }
        else if (code[*code_n] == 'F')
        {
            if (F_command() == -1)
            {
                break;
            }
        }
        else if (code[*code_n] == 'R')
        {
            R_command();
        }
        else if (code[*code_n] == 'C')
        {
            C_command();
        }
        (*code_n)++;
        
    }
    
    *code_n = temp;

    while (code[*code_n] != ']')
    {
        if (code[*code_n] == '[')
        {
            loop_command(code_n, code);
        }
        else if (code[*code_n] == 'F')
        {
            if (F_command() == -1)
            {
                break;
            }
        }
        else if (code[*code_n] == 'R')
        {
            R_command();
        }
        else if (code[*code_n] == 'C')
        {
            C_command();
        }
        (*code_n)++;
    }
    return 0;
}

int main()
{
    char file_name[5000];
    printf("Enter the output filename: ");
    fgets(file_name, 5000, stdin);
    file_name[strlen(file_name) - 1] = '\0';
    FILE *file = fopen(file_name, "wb");
    if (file == NULL)
    {
        printf("No file.\n");
        return -1;
    }
    printf("Enter the background color (R,G,B): ");
    sBmpPixel24 bg_color;
    scanf("(%hhd,%hhd,%hhd)", &bg_color.r, &bg_color.g, &bg_color.b);
    printf("Type code here: ");
    fgetc(stdin);
    char code[1000];
    fgets(code, 1000, stdin);
    code[strlen(code) - 1] = '\0';

    create_bmp24_bg(WIDTH, HEIGHT, bg_color, file);
    fclose(file);
    file = fopen(file_name, "rb");
    sBmpHeader header = read_header(file);
    // forget in midterm.
    for(int32_t i =0;i<256;i++)
    {
        for(int32_t j = 0;j<256;j++)
        {
            picture[i][j]=read_pixel(file,header,i,j,0);
        }
    }
    // forget in midterm
    file = fopen(file_name, "wb");

    {
        color_draw[0].r = 0xff;
        color_draw[0].g = 0xff;
        color_draw[0].b = 0xff;

        color_draw[1].r = 0;
        color_draw[1].g = 0;
        color_draw[1].b = 0;

        color_draw[2].r = 0x33;
        color_draw[2].g = 0x66;
        color_draw[2].b = 0xff;

        color_draw[3].r = 0x00;
        color_draw[3].g = 0xcc;
        color_draw[3].b = 0x00;

        color_draw[4].r = 0x00;
        color_draw[4].g = 0xcc;
        color_draw[4].b = 0xcc;

        color_draw[5].r = 0xcc;
        color_draw[5].g = 0x00;
        color_draw[5].b = 0x00;

        color_draw[6].r = 0xcc;
        color_draw[6].g = 0x00;
        color_draw[6].b = 0xcc;

        color_draw[7].r = 0xcc;
        color_draw[7].g = 0xcc;
        color_draw[7].b = 0x00;
    }
    int32_t code_n = 0;
    while (code[code_n] != '\0')
    {
        if (code[code_n] == '[')
        {
            loop_command(&code_n, code);
        }
        else if (code[code_n] == 'F')
        {
            if (F_command() == -1)
            {
                break;
            }
        }
        else if (code[code_n] == 'R')
        {
            R_command();
        }
        else if (code[code_n] == 'C')
        {
            C_command();
        }
        code_n++;
    }

    fwrite(&header, sizeof(sBmpHeader), 1, file);
    for (int i = 0; i < HEIGHT; i++)
    {
        for (int j = 0; j < WIDTH; j++)
        {
            fwrite(&picture[i][j], sizeof(sBmpPixel24), 1, file);
        }
        write_edge_pixel(WIDTH, file);
    }
    fclose(file);
    return 0;
}
```

## 結論

這次因為最後剩十幾分鐘來不及寫完第一題，不過如果有個一小時應該是寫得完第一題的，然後第四題是真的好玩，感謝老師調降難度。