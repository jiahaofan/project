#include<stdio.h>
#include<assert.h>
typedef enum ConvertState
{
	SUCCESS,
	FILE_ERROR,
	NO_MATCH,
}ConvertState;
typedef enum State
{
	C_BEGIN,
	C_END,
};
ConvertState Convert(FILE *fIn, FILE *fOut)
{
	assert(fIn);
	assert(fOut);
	int flag;
	char first, second;	
	flag = C_BEGIN;
	do
	{
		
		first = fgetc(fIn);
		switch (first)
			
		{
			// 一般情况
		
			case '/':
				second = fgetc(fIn);

				//匹配问题
				
				if (second == '*'&&flag == C_END)
				{
					fputc('/', fOut);
					fputc('/', fOut);
					flag = C_BEGIN;

				}		
				//c++注释问题
				else if (second == '/')
				{
					char Next = fgetc(fIn);
					fputc('/', fOut);
					fputc('/', fOut);
					while (Next != '\n'&&Next != EOF)
					{
						fputc(Next, fOut);
						Next = fgetc(fIn);
					}
					
					fseek(fIn, -1, SEEK_CUR);
				}
				else
				{
					fputc(first, fOut);
					fputc(second, fOut);
				}
				break;
			case '*':

				second = fgetc(fIn);

				//换行问题连续注释问题
				
				if (second == '/')
				{
					char Next = 0;
					Next = fgetc(fIn);
					if (Next != '\n'&&Next != EOF)
					{
						fseek(fIn, -1, SEEK_CUR);
						fputc('\n', fOut);
					}
					else
					{
						fputc('\n', fOut);
					}
					flag = C_END;
				}
				else if (second == '*')
				{
					fputc(first, fOut);
					fseek(fIn, -1, SEEK_CUR);
					break;
				}
				else
				{
					fputc(first, fOut);
					fputc(second, fOut);
				}
				break;
				//多行注释问题
			case '\n':
				fputc('\n', fOut);
				if (flag == C_BEGIN)
				{
					fputc('/', fOut);
					fputc('/', fOut);
				}
				break;
			default:
				fputc(first, fOut);
				break;
		}
	} while (first!= EOF);
	if (flag == C_END)
		return 	SUCCESS;
	else
		return 	NO_MATCH;
}




#define _CRT_SECURE_NO_WARNINGS

#include"conver.h"
int main()
{
	FILE *in;
	FILE *out;

	in=fopen("fIn.txt", "r");
	out=fopen("fOut.txt", "w");
	if (in == NULL)
	{
		fclose(out); 
		fclose(in);
		printf("打开文件失败\n");


	}
	int ret= Convert(in, out);
	if (ret == NO_MATCH)
	{
		printf("不匹配\n");
	}
	if (ret == SUCCESS)
	{
		printf("转换成功\n");
	}
	fclose(in);
	fclose(out);
	system("pause");
	return 0;
}