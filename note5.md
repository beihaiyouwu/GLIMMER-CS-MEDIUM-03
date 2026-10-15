# step1
### 1.1
1. 
* 绝对路径是真实完整的路径，有磁盘根目录，一级一级往下，会从根目录一级一级往下，一个文件夹一个文件夹的找，相当精确。
* 相对路径是从当前文件所在文件夹开始找，该文件只能存在于当前文件夹，否则就找不到。如果在上一级文件夹可以使用..\向上跳一级查找，./就代表的是当前文件夹，本质是个相对路径。
2. 绝对路径写法 cd /var/backups/turing/，相对路径写法 cd ../../../var/backups/turing/,一步一步返回上一级目录，在往下,也可以写成cd ../../var/backups/turing/，因为../可以跨越到同级别的根目录下，home和val是同级别的根目录。
### 1.2
1. argc和argv是main的参数，用来接收命令行的参数。argc代表参数个数，包含了程序名。argv[]是一个指针字符串数组，其中每个元素指向一个字符串，argv[argc]=NULL，用来表示数组结尾。
2. 如果是命令行输入是/var/backups/turing/，则argc=3，因为包含了程序名，argv[0]="/var"，argv[1]="/backups",argv[2]="/turing"。
# step2
### 2.1
1. 
* <stdio.h><string.h><stdlib.h>见过。第一个是几乎每个程序都有的，包含了常见的打印输入等库函数。第二个主要用在读取字符串长度，包括strlen和sizeof。第三个主要是在写链表二叉树时见，用于malloc分配内存和free释放内存。
* <dirent.h><sys/stat.h><sys/type.h><time.h>基本没见过。
2. 
* <dirent.h>用于读取目录，遍历子目录和文件。包含两个重要结构体DIR（表示目录流，拿到读整个文件夹的通道）和struct dirent（存单个文件信息），包含d_name存文件名，d_ino存文件的编号和d_type存文件类型。核心函数有三个，opendir打开目录，参数传目录路径（绝对路径和相对路径都行）。readdir读取目录下的下一个文件夹或文件，直到目录末尾返回NULL，每次readdir都会覆盖内容。closedir表示关闭文件或文件夹。
* <sys/sta.h>作用是获取文件目录的属性信息，通过读取它的路径，找到它的大小，权限，修改时间等属性内容，和dirent.h搭配使用。dirent.h遍历读到文件名后拿给stat读取详细属性。核心结构体struct stat，包含st_mode文件类型权限，st_size文件大小，st_mtime修改时间等重要内容，存的就是文件属性。核心函数stat（），参数传文件路径和stat结构体指针，作用是拿到文件路径，读取并将相关信息填充在stat中。fstat传的参数是open打开后的文件描述符（一个整数编号，便于程序间沟通），还有一个lstat作用和stat差不多，唯一不同在于如果程序里有符号链接，stat会跑到链接所在的其他文件夹，而lstat不会，优先用lstat。拿到 st_mode 后，可以用相关宏进行判断。S_ISREG(m)：普通文件，S_ISDIR(m)：目录，S_ISLNK(m)：符号链。
* 两者相互配合，一个读拿文件，一个存文件，再将其打印出来，便实现ls相关功能。
### 2.2
1. 权限可以用1表示有，0表示无，而这恰好也符合二进制写法。rwx可以分别对应三位，例如101表示r和x权限有，w权限无。权限包含ugo三者，自己属主，同组和其他人，便可以用相应9位表示。在分别转化成八进制就可以表示权限了（一共有0到7这八种表示所以用八进制刚好）。首先一个八进制数字便可以表示这三种权限，例如101表示r-x，转化成八进制即5表示可读可执行。000代表得数字0表示---，三种权限都无。输入三个八进制数，便可表示ugo三者的相应权限。比如000代表三者rwx权限都无，710表示u三种权限都有，g只有x权限，o三者都无，表示出来就是rwx--x---。通过这样便实现了用整数表示权限。当然除了用整数表示，也可以直接用符号+-=表示权限的增加减少和设置，具体格式为chmod u+r 文件 表示给自己增加r权限。
2. 
```c
// mode即传入的存储权限信息的整数， 如0754（八进制表示）
// 提示：传入的 mode 变量不仅包含权限信息，高位可能还包含文件类型。在进行位运算时，请注意屏蔽（Mask）掉不需要的高位数据。
void parse_permissions(mode_t mode, char *perms) {

    // 权限字符对照表
    char rwx[] = {'r', 'w', 'x'};

    // 请补全代码写在 TO DO 与 END OF TO DO 之间：
    // TO DO
    mode=mode&((1<<9)-1);//屏蔽高位
    for(int i=8;i>=0;i--)
    {
      if(((mode>>i)&1)==1)//取二进制的第i+1位判断，如果是1说明有权限
      {
        if(i%3==2)//第i+1位是3的倍数说明是r
        perms[8-i]=rwx[0];//位越高数组中存的位置越低
        if(i%3==1)
        perms[8-i]=rwx[1];
        if(i%3==0)
        perms[8-i]=rwx[2];
      } 
      else
      perms[8-i]='-'; //不是1无权限保留'-'
    }
    // END OF TO DO

    perms[9] = '\0'; // 字符串结尾
}
```
* 首先传过来的是个八进制数，同时高位还有其它数字需要屏蔽，才用（1<<9）-1屏蔽掉了除后9位其余的。这之后便要对二进制逐位判断了，用1来取。先取最高位，再不断移动取低位，如果是1说明有权限。同时还有判断是rw还是x，就用它是第几位判断，比如369位就是r，258就是w，147就是x，依次把他们存在perms数组里就行了。为了打印顺序对，二进制中高位的存在数组中地位，方便从小到大打印。
* 为了验证是否正确，我添加了打印数组的循环进行验证，我将mode设置为750。
* 输出如下 

![alt text](QQ20261014-142308.png)
# step3
### 3.1
1. inode存储文件的属性，具体包括文件的大小，类型，权限，修改阅读时间等。除此之外还包括指向数据块的指针。
2. inode不包含文件名和文件内容，文件名存在目录中，文件内容存在数据块中，要用fopen和fread才能读，而索引节点就是属性单。每个目录对应一个inode编号，通过编号找到索引节点，即可查阅相关文件信息，再通过数据块指针找到数据块拿到文件内容。
3. "."代表当前目录的inode编号，".."代表上一级目录的inode编号。特殊的在于根目录下两者的编号相同，通常是2。根目录已经是树的最高点根部起点了，没有父目录，认为规定两者编号相同且为2，至于为什么不是0在于0和1都有其它的用途。
### 3.2
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>

ino_t get_inode(const char *path) {
    struct stat statbuf;//存放获取的相关文件信息
    if (stat(path, &statbuf) == -1) //读取path对应的文件信息存到statbuf中
    {
        perror("stat failed");//出错打印出错原因
        exit(EXIT_FAILURE);//退出
    }
    return statbuf.st_ino;//返回一个编号
}
//当前目录下找到对应文件名
char* findname(ino_t target)
{
    DIR *dir=opendir(".");//打开当前目录得到目录流指针dir
    if(dir==NULL)//打开目录失败返回NULL
    {
        perror("opendir");
        exit(EXIT_FAILURE);
    }
    struct dirent *entry;//dirent结构体代表目录中的每一个文件子目录
    static char buf[256];
    //readdir循环读取目录里的每一项，读到NULL代表读完所有目录项
    while((entry=readdir(dir))!=NULL)
    {
        if(entry->d_ino==target)//读到目标编号
        {
            strcpy(buf,entry->d_name);//将d_name中的内容拷贝到bug中
            closedir(dir);//关闭目录流
            return buf;//返回目录字符串
        }
    }
    closedir(dir);
    return NULL;
}
int main(void)
{
    //栈数组，保存一层层向上找到的目录名字
    char path[100][100];
    int top=0;//栈顶下标，记录存了多少个目录名
    ino_t cur=get_inode(".");//获取当前工作目录的inode
    ino_t parent;
    for(;;)//无线循环，一直往父目录走，直到根目录
    {
        if(chdir("..")==-1)//切换工作目录到父目录
        {
            perror("chdir ..");
            exit(EXIT_FAILURE);
        }
        //已经到父目录，获取父目录的inode编号
        parent=get_inode(".");
        if(cur==parent)//当前目录编号等于父节点标号，说明到达根节点
        {
            break;
        }
    
    char *name=findname(cur);//把找到的目录压入栈中
    strcpy(path[top++],name);
    cur=parent;//把父inode作为上一层的子inode，不断向上。
    }
    //逆序打印
    putchar('/');//路径最开始是根斜杠
    for(int i=top-1;i>=0;i--)//最后存的是最上面的，倒序循环输出
    {
        printf("%s",path[i]);
        if(i>0)
        putchar('/');
    }
    putchar('\n');return 0;

}
```
* 整体思路：首先我觉得这个程序可以分为四个部分，第一个获取inode编号，即get_inode函数。第二个是根据inode找到文件名，即findname函数。第三个是实现有当前目录到根目录的由下到上获得当前文件名的过程，最后一个是从上到下打印这些文件名，这两个部分都在主函数中。首先要获得inode编号我们需要获得一个路径，利用stat函数将这个文件的各种信息存在statbuf结构体中，在这里我们只需要我们只需要返回一个编号就可以了。接着是根据编号找名字，这里我们必须是在父目录去找相应子目录，所以主函数中要使用chdir跳到上一级目录。接着我们创建一个dirent结构体以获取名字，具体步骤为使用opendir打开当前目录，保存在dir目录流中，再使用readdir依次读取里面的文件的编号，读到编号就把把文件名保存拷贝给字符串数组buff，然后返回给主函数buff，这样我们就得到了文件名。下一部分是从下往上的这样一个过程，我们需要得到子目录和父目录的相应节点，还需要一个数组来保留这些文件名，每次就把拿到的这个文件名保存在这个数组中。关键在于终点的判断，如果此时当前目录和父目录的编号一样则说明到达根。为实现从下到上的过程，循环最后需要将父编号赋给子编号，开启下一轮循环。最后一个部分是打印，这个像栈，最底下的最先进去最后打印，先进后出，根最后进得最先打印，把数组从后往前打印就可以了。
* AI使用说明；对于AI使用我主要是在怎么得到文件名这个函数中，由于我最开始不知道是要在上一级目录中中通过inode找寻文件，导致我一直卡在当前文件中，为此我像AI阐明相关问题，寻求相关帮助。
* prompt
![alt text](QQ20261015-233353.png)

# step4
```c
#include <stdio.h>
#include <string.h>
#include <dirent.h>
#include <sys/stat.h>
#include <unistd.h>

// 递归查找指定文件，若找到则打印其相对路径
void find_file(const char *current_path, const char *target_name) {
    DIR *dir;
    struct dirent *entry;
    struct stat statbuf;

    if (!(dir = opendir(current_path))) {
        return; // 无法打开目录则跳过
    }

    while ((entry = readdir(dir)) != NULL) {
        // 忽略当前目录 . 和父目录 .. 防止无限死循环
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0) {
            continue;
        }

        char full_path[1024];
        snprintf(full_path, sizeof(full_path), "%s/%s", current_path, entry->d_name);

        if (stat(full_path, &statbuf) == -1) continue;

        // 请补全代码写在 TO DO 与 END OF TO DO 之间：
        // 要求：
        // 1. 检查当前 entry 的文件名是否与 target_name 匹配，若匹配则打印 full_path。
        // 2. 如果当前 entry 是一个目录，则需要对其进行深度优先的递归查找。
        // （提示：若(statbuf.st_mode & S_IFMT) == S_IFDIR则statbuf为目录文件）
        // TO DO
        if(strcmp(entry->d_name,target_name)==0)//比较字符串内容，一致则等于0
        {
            printf("%s",full_path);//打印该路径
        }
        if((statbuf.st_mode&S_IFMT)==S_IFDIR)//如果是目录
        {
            find_file(full_path,target_name);//递归下一级目录
        }



        // END OF TO DO
    }
    closedir(dir);
}
```
* 为了找到目标文件，我们首先需要打开当前目录去一个一个读，如果当前目录就能读到目标文件名，就能直接打印。如果还有子目录就要递归到子目录中去寻找。注意这里要用strcmp函数来比较相应字符串的内容，如果一样就返回0，代表找到了就直接打印，没有则继续循环。
