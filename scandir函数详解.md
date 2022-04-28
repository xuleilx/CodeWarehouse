# 头文件
```c
#include <dirent.h>
```
# 函数定义
```c
int scandir(const char *dir, struct dirent ***namelist,
			int (*select)(const struct dirent *),
			int (*compar)(const struct dirent **, const struct dirent **));
```
# 函数说明
`scandir()`会扫描参数dir指定的目录文件，经由参数select（转者注：通过Linux的man手册查看函数原型，该形参名为fliter（过滤）可能目的更明确）指定的函数来挑选目录结构至参数namelist数组中，最后再调用参数compar指定的函数来排序namelist数组中的目录数据。每次从目录文件中读取一个目录结构后便将此结构传给参数select所指的函数，select函数若不想要将此目录结构复制到namelist数组就返回0，若select为空指针则代表选择所有的目录结构。scandir()会调用qsort()来排序数据，参数compar则为qsort()的参数，若是要排列目录名称字母则可使用alphasort()。结构dirent定义请参考readdir()。

（转者注：结构体dirent的定义最终在/usr/include/x86_64-linux-gnu/bits/dirent.h文件中找到）
```c
struct dirent {
#ifndef __USE_FILE_OFFSET64
    __ino_t d_ino; 				/* inode number 索引节点号 */
    __off_t d_off; 				/* offset to this dirent 在目录文件中的偏移 */
#else
    __ino64_t d_ino;
    __off64_t d_off;
#endif	
	unsigned short	d_reclen; 	/* length of this d_name 文件名长 */
	unsigned char	d_type; 	/* the type of d_name 文件类型 */
	char			d_name[256];/* file name (null-terminated) 文件名，最长255字符 */
};
```
返回值
成功则返回复制到namelist数组中的数据结构数目，有错误发生则返回-1。

# 测试代码如下
## example 1
```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
 
int main(void)
{
    struct dirent **entry_list;
    int count;
    int i;
 
    count = scandir("/home/book/workspace", &entry_list, 0, alphasort);
    if (count < 0) {
        perror("scandir");
        return EXIT_FAILURE;
    }
    for (i = 0; i < count; i++) {
        struct dirent *entry;
        entry = entry_list[i];
        printf("%s\n", entry->d_name);
        free(entry);
    }
    free(entry_list);
 
    return 0;
}
```
## example 2
（转者注：为避免程序编译警告，程序已作修改，对自定义函数scan参数类型及返回值进行修正。）
```c
#define _GNU_SOURCE 	/* 使用versionsort需要加上该宏定义 */
 
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <dirent.h>
 
void make_test_data(char *path)
{
    int i;
 
    for (i = 0; i < 20; i++) {
        char file[128];
        sprintf(file, "file-%d", i);
        creat(file, 0);
    }
}
void scan(char *path, char *title, int (*sort)(const struct dirent **, const struct dirent **))
{
    struct dirent **entry_list;
    int count;
    int i;
    puts(title);
    count = scandir(path, &entry_list, 0, sort);
    if (count < 0) {
        perror("scandir");
        return;
    }
    for (i = 0; i < count; i++) {
        struct dirent *entry;
        entry = entry_list[i];
        printf("%s\n", entry->d_name);
        free(entry);
    }
 
    printf("\n");
    free(entry_list);
}
 
int main(void)
{
    char *path = ".";
 
    make_test_data(path);
    scan(path, "alphasort: ", alphasort);
    scan(path, "versionsort: ", versionsort);
    return 0;
}
```
## example 3
```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

int filter(const struct dirent *entry)
{
    return entry->d_name[0] == 'a';
}

int main(void)
{
    struct dirent **entry_list;
    int count;
    int i;

    count = scandir("/usr/include", &entry_list, filter, alphasort);
    if (count < 0) {
        perror("scandir");
        return EXIT_FAILURE;
    }
    for (i = 0; i < count; i++) {
        struct dirent *entry;
        entry = entry_list[i];
        printf("%s\n", entry->d_name);
        free(entry);
    }
    free(entry_list);

    return 0;
}
```
