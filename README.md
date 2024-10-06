#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef struct{
    char name;
    int rows,cols;
    int isempty;
    /*I try to decide true or false.
    I search online,and learn the 'boolean'.
    Because I stil don't understand how to use 'boolean',i judge right and wrong by changing the value of a variable*/
    double **my2DArray;
}m, Matrix;
Matrix matrix[26];
double **Array2D(size_t Rows,size_t cols)
{
    double **p = (double **)malloc(Rows*sizeof(double *)+
                                    +Rows*cols*sizeof(double *));
    if(p==NULL)
    {
        printf("Memory allocation error");
    }
    for(int i=0;i<Rows;i++)
        p[i] = (double *)(p + Rows) + i*cols;
        
    return p;
}
/* Learn from iLearnig 3.0*/
int getnumber(FILE *file,char c)
{
    int i=0,value;
    char str[20] = {},hashtag = 0;
    while(c < '0' || c >'9' || hashtag ==1){
        if(c == '\n' || c == '\r'){
            hashtag=0;
        }
        else if(c == '#'){
            hashtag=1;
        }
        c=getc(file);
    }
    do{
        str[i]=c;
        i++;
        c = getc(file);
    } while(c != ' ' && c != '\n' && c != '\r');
    ungetc(c,file);
    /*learn from https://www.cplusplus.com*/
    value=atoi(str);
    return value;
}
void print_Array(Matrix m)
{
    printf("\n%c %d %d",m.name,m.rows,m.cols);
    for(int x=0;x<m.rows;x++){
        printf("\n");
        for(int y=0;y<m.cols;y++){
            printf("%15.5e\t",m.my2DArray[x][y]);
        }
    }
    printf("\n");
}
int findindex(char name)
{
    return name-'A';
}
/* learn from iLearningFinalExam_v1.3 Instruction*/ 
Matrix add(Matrix m1,Matrix m2)
{
    Matrix M;
    M.rows=m1.rows;
    M.cols=m1.cols;
    M.my2DArray = Array2D(M.rows,M.cols);
    if(M.my2DArray == NULL){
        printf("Memory allocation error.\n");
    }
    if(m1.rows != m2.rows || m1.cols != m2.cols){
        printf("Matrix size imcompatible.\n");
    }
    else{
        for(int x=0;x<m1.rows;x++){
            for(int y=0;y<m1.cols;y++){
                M.my2DArray[x][y]=m1.my2DArray[x][y]+m2.my2DArray[x][y];
            }
        }
    }
    return M;
}
Matrix sub(Matrix m1,Matrix m2)
{
    Matrix M;
    M.rows=m1.rows;
    M.cols=m1.cols;
    M.my2DArray = Array2D(m1.rows,m1.cols);
    if(M.my2DArray == NULL){
        printf("Memory allocation error.\n");
    }
    if(m1.rows != m2.rows || m1.cols != m2.cols){
        printf("Matrix size imcompatible.\n");
    }
    else{
        for(int x=0;x<m1.rows;x++){
        for(int y=0;y<m1.cols;y++){
            M.my2DArray[x][y]=m1.my2DArray[x][y]-m2.my2DArray[x][y];
            }
        }
    }
    return M;
}
Matrix mul(Matrix m1,Matrix m2)
{
    Matrix M;
    int sum;
    M.rows=m1.rows;
    M.cols=m2.cols;
    M.my2DArray = Array2D(m1.rows,m2.cols);
    if(M.my2DArray == NULL){
        printf("Memory allocation error.\n");
    }
    if(m1.rows != m2.cols || m1.cols != m2.rows){
        printf("Matrix size imcompatible.\n");
    }
    else{
        for(int i=0;i<m1.rows;i++){
        for(int j=0;j<m2.cols;j++){
            sum=0;
            for(int k=0;k<m1.cols;k++){
                sum=sum+m1.my2DArray[i][k]*m2.my2DArray[k][j];
                M.my2DArray[i][j]=sum;
            }
        }
    }
    M.isempty=1;
    }
    return M;
}
void read_data(char *filename)
{
    char c;
    int hashtag=0,letter=0,i;
    FILE *pfile = fopen(filename,"r");
    /* learn from iLearningFinalExam_v1.3.pdf Instruction*/ 
    if(pfile == NULL){
        printf("File error.\n");
    }
    while( (c = getc (pfile)) != EOF){
        if(c == '\n'||c == '\r'){
            hashtag=0;
        }
        else if(c == '#'){
            hashtag=1;
        }
        else if(hashtag != 1){
            if(c <= 'Z' && c >= 'A'){
                letter=1;
                i=findindex(c);
                matrix[i].name = c;
                matrix[i].isempty=1;
            }
            else if(letter == 1){
                matrix[i].rows = getnumber(pfile,c);
                matrix[i].cols = getnumber(pfile,c);
                matrix[i].my2DArray=Array2D(matrix[i].rows,matrix[i].cols);
                if(matrix[i].my2DArray == NULL){
                    printf("Memory allocation error.\n");
                }
                for(int x=0;x<matrix[i].rows;x++){
                    for(int y=0;y<matrix[i].cols;y++){
                        matrix[i].my2DArray[x][y]=getnumber(pfile,c);
                    }
                }
                letter=0;
            }    
        }
    }   
}
int main()
{
    char order[128]={},order1[128]={},c1,c2;
    int i,a,b,c,f_rows,l_rows,f_cols,l_cols,cnt,m,x,y;
    do{
        printf("$ ");
        scanf("%s", order);
        /*because i use string to store the command ,there is not space in the command */
        if(order[0] == '<' || order[0] == '>'){
            memcpy(order1,&order[1],strlen(order)-1);
            /* learn from http://tw.gitbook.net/c_standard_library/c_function_memcpy.html*/
            read_data(order1);
        }
        else if(order[0] == '?'){
            if(order[1] >= 'A' && order[1] <= 'Z'){
                i=findindex(order[1]);
                if(matrix[i].isempty != 1){
                    printf("Matrix not exist.\n");
                }
                else{
                    print_Array(matrix[i]);
                }
            }
            else{
                printf("Matrixes status:\n");
                printf("Name\trols\tcols\n");         
                for(i=0;i<=26;i++){
                    if(matrix[i].isempty == 1){
                        printf("%c\t%d\t%d\n",matrix[i].name,matrix[i].rows,matrix[i].cols);
                    }
                }
            }
        }
        else if(order[0] == '!' && order[1] <= 'Z' && order[1] >= 'A'){
            i=findindex(order[1]);
            matrix[i].isempty=0;
        }
        else if(strcmp(order,"!!d") == 0){
            /* learn from https://cplusplus.com/reference/cstring/strcmp/?kw=strcmp*/
            for(int j=0;j<=26;j++){
                matrix[j].isempty=0;
            }
        }
        else if(order[0] <= 'Z' && order[0] >= 'A'){
            if(order[3] == '+'){
                a=findindex(order[0]);
                b=findindex(order[2]);
                c=findindex(order[4]);
                if(matrix[b].isempty != 1 || matrix[c].isempty != 1){
                    printf("matrix operation is not exist\n");
                }
                else{
                    matrix[a]=add(matrix[b],matrix[c]);
                    matrix[a].name=order[0];
                    matrix[a].isempty=1;
                }
            }
            else if(order[3] == '-'){
                a=findindex(order[0]);
                b=findindex(order[2]);
                c=findindex(order[4]);
                if(matrix[b].isempty != 1 || matrix[c].isempty != 1){
                    printf("matrix operation is not exist\n");
                }
                else{
                    matrix[a]=sub(matrix[b],matrix[c]);
                    matrix[a].name=order[0];
                    matrix[a].isempty=1;
                }
            }
            else if(order[3] == '*'){
                a=findindex(order[0]);
                b=findindex(order[2]);
                c=findindex(order[4]);
                if(matrix[b].isempty != 1 || matrix[c].isempty != 1){
                    printf("matrix operation is not exist\n");
                }
                else{
                    matrix[a]=mul(matrix[b],matrix[c]);
                    if(matrix[a].isempty == 1){
                        matrix[a].name=order[0];
                    }
                }
            }
            else if(order[2] == '+'){
                a=findindex(order[0]);
                b=findindex(order[3]);
                matrix[a].my2DArray=matrix[b].my2DArray;
                matrix[a].name=order[0];
                matrix[a].isempty=1;
            }
            else if(order[2] == '-'){
                a=findindex(order[0]);
                b=findindex(order[3]);
                matrix[a].rows=matrix[b].rows;
                matrix[a].cols=matrix[b].cols;
                matrix[a].my2DArray=Array2D(matrix[a].rows,matrix[a].cols);
                for(int k=0;k<matrix[a].rows;k++){
                    for(int j=0;j<matrix[a].cols;j++){
                        matrix[a].my2DArray[k][j]=-matrix[b].my2DArray[k][j];
                    }
                }
                matrix[a].name=order[0];
                matrix[a].isempty=1;
            }
            else if(order[3] == '['){
                m=sscanf(order,"%c=%c[%d:%d,%d:%d]%n",&c1,&c2,&f_rows,&l_rows,&f_cols,&l_cols,&cnt);
                /* learn from iLearningFinalExam_v1.3 Instruction*/                
                a=findindex(c1);
                b=findindex(c2);
                x=0;
                y=0;
                if(l_rows > matrix[b].rows || l_cols >matrix[b].cols){
                    printf("Matrix out of index.\n");
                }
                else{
                    matrix[a].rows=l_rows-f_rows;
                    matrix[a].cols=l_cols-f_cols;
                    matrix[a].my2DArray=Array2D(matrix[a].rows,matrix[a].cols);
                    for(int k=f_rows;k<l_rows;k++){
                        y=0;
                        for(int j=f_cols;j<l_cols;j++){
                            matrix[a].my2DArray[x][y]=matrix[b].my2DArray[k][j];
                            y++;
                        }
                        x++;
                    }
                    matrix[a].name=order[0];
                    matrix[a].isempty=1;
                }
            }
        }
        else if(strcmp(order,"!!q") == 0){
            for(i=0;i<26;i++){
                free(matrix[i].my2DArray); 
            }
        }
        else{
            printf("commend error.\n");
        }
    }while(strcmp(order,"!!q") != 0);
}
