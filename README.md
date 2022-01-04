int load_data()
{
    FILE* f1;
    FILE* f2;
    FILE* f3;
    FILE* f4;
    f1 =fopen("\ROUTES.txt","r");
    f2 =fopen("BUSES.txt","r");
    f3 =fopen("STATIONS.txt","r");
    f4 =fopen("MAPNUM.txt","r");
    if(f1==NULL)
    {
        printf("load ROUTES.txt errer!\n");
        exit(0);
    }
    else if(f2==NULL)
    {
        printf("load BUSES.txt errer!\n");
        exit(0);
    }
    else if(f3==NULL)
    {
        printf("load STATIONS.txt errer!\n");
        exit(0);
    }
    else if(f4==NULL)
    {
        printf("load MAPNUM.txt errer!\n");
        exit(0);
    }
    else
    {

        fscanf(f4,"%d%d%d",&BUS_NUM,&STATION_NUM,&ROUTE_NUM);
        fclose(f4);
        ROUTES= malloc(sizeof(int)*ROUTE_NUM * 4);
        BUSES= malloc(sizeof(char*)*BUS_NUM* 3);
        STATIONS= (char**)malloc(sizeof(char*)*STATION_NUM);

        int i;
        for(i=0;i<ROUTE_NUM;i++)
        {
            fscanf(f1,"%d%d%d%d",&ROUTES[i][0],&ROUTES[i][1],&ROUTES[i][2],&ROUTES[i][3]);
        }
        for(i=0;i<BUS_NUM;i++)
        {
            BUSES[i][0]= (char*)malloc(20*sizeof(char));
            BUSES[i][1]= (char*)malloc(20*sizeof(char));
            BUSES[i][2]= (char*)malloc(20*sizeof(char));
            fscanf(f2,"%s%s%s",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
        }
        for(i=0;i<STATION_NUM;i++)
        {
            STATIONS[i]= (char*)malloc(20*sizeof(char));
            fscanf(f3,"%s",STATIONS[i]);
        }

    }
    fclose(f1);
    fclose(f2);
    fclose(f3);
    return 0;
}


//4.2 采用邻接表储存公交线路图：

//通过void LoadMapDate()函数把数组的数据载入全局变量g_sMap中。
typedef struct Bus
{
    char* name; //公交线路名
    int start;  //起点
    int end;    //终点
    int* stations;   //公交线路站点索引数组
    int station_num;//站点个数
}Bus;

//2、定义结构体STATION代表一个站点

typedef struct Station
{
    char* station;          //站点名
    struct Route* routes;   //从该站点出发的所有下行路线的链域
}Station;

//4.3定义结构体ROUTE代表公交线路中的一个路段（邻接表结点）

typedef struct Route
{
    int station;           //指向的站点索引号
    int bus;               //公交索引号
    int distance;          //两点之间公路的距离
    bool visited;          //遍历时的标识符
    struct Route* next;    //起始站点相同的，下一条下行路线
}Route;


//4.4定义结构体BUSMAP存储整个公交地图信息

typedef struct BusMap
{
    Bus* buses;             //公交线路数组
    Station* stations;       //站点数组
    int station_num;        //站点数
    int bus_num;            //公交线路数
}BusMap;


BusMap g_sMap;  //定义全局变量


//4.5查询公交线路 
//通过QueryBus函数查询

int QueryBus(char* pBus,char** stations)
{
    int sum=2;      //记录站点个数,起点和终点没算
    //1、查找公交
    int flagBus=FindBus(pBus);    //找到公交线路的索引
    //2、找到公交及信息
    char* nStart=BUSES[flagBus][1];
    char* nEnd=BUSES[flagBus][2];
    //3、输出起始站点
    printf("[%s线路]\t从[%s]开往[%s]\n",pBus,nStart,nEnd);
    //4、输出各站点
    int flagStart = FindStation(nStart);    //找到起点的索引，从而找到后面路线
    int flagEnd = FindStation(nEnd);
    Station* pStStation = &g_sMap.stations[flagStart];
    Route* pStRoute = pStStation->routes;

    while(pStRoute->bus!=flagBus)       //找到该公交线路的第一条路线
    {
        pStRoute=pStRoute->next;
    }

    printf("%s->",nStart);       //输出起始站点

    while(pStRoute->station!=flagEnd)       //如果路线不是最后一条路线
    {
        printf("%s->",*(stations+(pStRoute->station)));
        sum++;
        pStStation = &g_sMap.stations[pStRoute->station];    //换到下一个路线的站点
        pStRoute = pStStation->routes;     //下一个站点的第一条路线，不是该公交线路的第一条路线
        while(pStRoute->bus!=flagBus)
        {
            pStRoute=pStRoute->next;
        }
    }
    printf("%s\n",nEnd);
    return sum;
}

//4.6查询站点信息 
//函数代码如下：

//创建QueryStation函数，实现查询站点信息，输出该站点所经线路信息

int QueryStation(char* pStation,char** buses)
{
    int flag[BUS_NUM];
    int i;
    for(i=0;i<BUS_NUM;i++)
    {
        flag[i]=-1;
    }

    int sum=0;                     //记录路线的总数
    int flagStation=FindStation(pStation); //找到该站点的索引
    Station* pStStation=&g_sMap.stations[flagStation];
    Route* pStRoute=pStStation->routes;
    int j=0;
    while(pStRoute!=NULL)
    {
        flag[j]=pStRoute->bus;
        j++;
        pStRoute=pStRoute->next;
    }

    for(i=0;flag[i]!=-1;i++)
    {
        printf("%s\n",*(buses+(flag[i])*3));
        //printf("%s\n",*(buses+((flag[i])+1)*3));
        sum++;
    }
    return sum;
}
//函数代码如下：
void Show_transferPath(char* pStart,char* pEnd)
{
    int nStart = FindStation(pStart);
    int nEnd = FindStation(pEnd);
    int path_sum=0;
    int i=0,j=0;
    //先输出直达的路线
    int  pathdirect_bus[BUS_NUM];
    for(i=0;i<BUS_NUM;i++)
    {
        pathdirect_bus[i]=-1;
    }
    bool is_pathdirect=directPath(pStart,pEnd,pathdirect_bus);  //判断是否有直达，如果有，把直达公交放在数组中

    if(is_pathdirect==true)
    {
        for(i=0;pathdirect_bus[i]!=-1;i++)
        {
            path_sum++;
            printf("第%d条路线为：\n",path_sum);
            Show_directPath(pStart,pEnd,pathdirect_bus[i]);
            printf("\n\n");
        }
    }

    //换乘一次的路线

    int stations[STATION_NUM-2];    //记录每个站点的索引
    i=0;
    for(i=0;i<STATION_NUM;i++)
    {
        int station_flag = FindStation(STATIONS[i]);
        if(station_flag!=nStart&&station_flag!=nEnd)
        {
            stations[j]=station_flag;
            j++;
        }
    }

    int Start_directbus[BUS_NUM];
    int End_directbus[BUS_NUM];
    i=0;
    for(i=0;i<BUS_NUM;i++)
    {
        Start_directbus[i]=-1;
        End_directbus[i]=-1;
    }

    i=0;j=0;
    int m,n;
    for(i=0;i<(STATION_NUM-2);i++)
    {

        //必须要给数组重置
        for(j=0;j<BUS_NUM;j++)
        {
            Start_directbus[j]=-1;
            End_directbus[j]=-1;
        }

        bool Start_middle = directPath(pStart,STATIONS[stations[i]],Start_directbus);
        bool End_middle = directPath(STATIONS[stations[i]],pEnd,End_directbus);

        if(Start_middle==true&&End_middle==true)
        {
            for(m=0;Start_directbus[m]!=-1;m++)
            {
                for(n=0;End_directbus[n]!=-1;n++)
                {
                    if(Start_directbus[m]!=End_directbus[n])
                    {
                        path_sum++;
                        printf("第%d条路线为：\n",path_sum);
                        Show_directPath(pStart,STATIONS[stations[i]],Start_directbus[m]);
                        printf("(在此站换乘)\n");
                        Show_directPath(STATIONS[stations[i]],pEnd,End_directbus[n]);
                        printf("\n\n");
                    }
                }

            }

        }
    }
    if(path_sum==0)
    {
        printf("没有符合要求的路线！\n");
    }

}

//4.7通过函数modify把全局变量的数据修改，然后再写入文件中保存，
//最后调用load_data()函数和void LoadMapDate()函数把数组的数据再载入全局变量g_sMap中。
//修改某一条线路时，要动态设置数组，删一条线路，ROUTES_NUM就应该减1，ROUTES文档再重新写入，如果删除的站点是某一条线路的起点和终点，还需要修改BUSES文档。
//代码如下：
int modify_data()
{
    int ch;
    char name1[20],name2[20];
    int distance;
    int i;
    printf("1、修改站点名称\n");
    printf("2、修改线路距离\n");
    printf("3、删除某一公交站点\n");
    ch=getch()-48;
    switch(ch)
    {
    case 1:
        printf("请输入要修改的站点原名称：");
        scanf("%s",name1);
        printf("请输入修改后的站点名称：");
        scanf("%s",name2);
        int flag = FindStation(name1);
        STATIONS[flag]=name2;
        //LoadMapDate();          //重新加载
        FILE* f1=fopen("D:\\MyWorkSpace\\c\\公交线路图\\STATIONS.txt","w");
        if(f1==NULL)
        {
            printf("open error!\n");
            exit(0);
        }
        for(i=0;i<STATION_NUM;i++)
        {
            fprintf(f1,"%s   ",STATIONS[i]);
        }
        fclose(f1);
        for(i=0;i<BUS_NUM;i++)
        {
            //printf("%d\n",g_sMap.buses[i].start);
            if(flag==g_sMap.buses[i].start)
            {
                if(i%2==0)
                {
                    BUSES[i][1]=name2;
                    BUSES[i+1][2]=name2;
                }
                else
                {
                    BUSES[i][1]=name2;
                    BUSES[i-1][2]=name2;
                }
            }
        }
        f1=fopen("D:\\MyWorkSpace\\c\\公交线路图\\BUSES.txt","w");
        if(f1==NULL)
        {
            printf("open BUSES error!\n");
            exit(0);
        }
        else
        {
            for(i=0;i<BUS_NUM;i++)
            {
                fprintf(f1,"%s  %s  %s\n",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
            }
            fclose(f1);
        }
        printf("\n修改成功！\n");
        break;
    case 2:
        printf("请输入想要修改的(相邻)两个站点：\n");
        printf("站点一：");
        scanf("%s",name1);
        printf("站点二：");
        scanf("%s",name2);
        printf("请输入这两站之间的距离(单位：米)：");
        scanf("%d",&distance);
        int flag1 = FindStation(name1);
        int flag2 = FindStation(name2);
        for(i=0;i<ROUTE_NUM;i++)
        {
            if(ROUTES[i][1]==flag1)
            {
                if(ROUTES[i][2]==flag2)
                {
                    ROUTES[i][3]=distance;
                }
            }
            else if(ROUTES[i][1]==flag2)
            {
                if(ROUTES[i][2]==flag1)
                {
                    ROUTES[i][3]=distance;
                }
            }
        }
        LoadMapDate();      //重新加载
        FILE* f2 = fopen("D:\\MyWorkSpace\\c\\公交线路图\\ROUTES.txt","w");
        for(i=0;i<ROUTE_NUM;i++)
        {
            fprintf(f2,"%d  %d  %d  %d\n",ROUTES[i][0],ROUTES[i][1],ROUTES[i][2],ROUTES[i][3]);
        }
        fclose(f2);
        break;
    case 3:
        printf("请输入要删除哪条公交线路的哪个站点：\n");
        printf("请输入公交线路：");
        scanf("%s",name1);
        printf("请输入站点：");
        scanf("%s",name2);
        int flag3 = FindStation(name2); //站点索引
        int flag4 = FindBus(name1); //公交线路索引

        if(flag3==g_sMap.buses[flag4].start)
        {
            if(flag4%2==0)
            {
                int modify1=Find_ROUTES_1(flag4,flag3);
                int modstart = ROUTES[modify1][2];      //先记录更新的起点
                delete_ROUTES(modify1);
                load_data();
                int modify2=Find_ROUTES_2(flag4+1,flag3);
                delete_ROUTES(modify2);
                load_data();

                BUSES[flag4][1]=STATIONS[modstart];
                BUSES[flag4+1][2]=STATIONS[modstart];
                FILE* f4 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\BUSES.txt","w");
                if(f4==NULL)
                {
                    printf("open BUSES error!\n");
                    exit(0);
                }
                for(i=0;i<BUS_NUM;i++)
                {
                    fprintf(f4,"%s  %s  %s\n",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
                }
                fclose(f4);
            }
            else
            {
                int modify1=Find_ROUTES_1(flag4,flag3);
                int modstart = ROUTES[modify1][2];
                delete_ROUTES(modify1);
                load_data();
                int modify2=Find_ROUTES_2(flag4-1,flag3);
                delete_ROUTES(modify2);
                load_data();

                BUSES[flag4][1]=STATIONS[modstart];
                BUSES[flag4-1][2]=STATIONS[modstart];
                FILE* f4 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\BUSES.txt","w");
                if(f4==NULL)
                {
                    printf("open BUSES error!\n");
                    exit(0);
                }
                for(i=0;i<BUS_NUM;i++)
                {
                    fprintf(f4,"%s  %s  %s\n",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
                }
                fclose(f4);
            }
            load_data();
        }
        else if(flag3==g_sMap.buses[flag4].end)
        {
            if(flag4%2==0)
            {
                int modify1=Find_ROUTES_2(flag4,flag3);
                int modend = ROUTES[modify1][1];        //记录更新的终点
                //printf("路线索引=%d\n更新的终点索引=%d\n",modify1,modend);
                delete_ROUTES(modify1);
                load_data();
                int modify2=Find_ROUTES_1(flag4+1,flag3);
                delete_ROUTES(modify2);
                load_data();

                BUSES[flag4][2]=STATIONS[modend];
                BUSES[flag4+1][1]=STATIONS[modend];
                FILE* f4 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\BUSES.txt","w");
                for(i=0;i<BUS_NUM;i++)
                {
                    fprintf(f4,"%s  %s  %s\n",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
                }
                fclose(f4);
            }
            else
            {
                int modify1=Find_ROUTES_2(flag4,flag3);
                int modend = ROUTES[modify1][1];
                delete_ROUTES(modify1);
                load_data();
                int modify2=Find_ROUTES_1(flag4-1,flag3);
                delete_ROUTES(modify2);
                load_data();

                BUSES[flag4][2]=STATIONS[modend];
                BUSES[flag4-1][1]=STATIONS[modend];
                FILE* f4 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\BUSES.txt","w");
                for(i=0;i<BUS_NUM;i++)
                {
                    fprintf(f4,"%s  %s  %s\n",BUSES[i][0],BUSES[i][1],BUSES[i][2]);
                }
                fclose(f4);
            }
        }
        else
        {
            printf("请输入删除该站点所形成的新路线的距离是(单位:米)：");
            scanf("%d",&distance);
            if(flag4 % 2!=0)
            {
                flag4--;
            }
            if(flag4 % 2==0)
            {
                FILE* f3 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\ROUTES.txt","w");
                int modify=Find_ROUTES_2(flag4,flag3);
                ROUTES[modify][2]=ROUTES[modify+1][2];
                ROUTES[modify][3]=distance;
                for(i=0;i<ROUTE_NUM;i++)
                {
                    fprintf(f3,"%d  %d  %d  %d\n",ROUTES[i][0],ROUTES[i][1],ROUTES[i][2],ROUTES[i][3]);
                }
                fclose(f3);
                delete_ROUTES(modify+1);
                load_data();        //把修改的文件再导入

                f3 =  fopen("D:\\MyWorkSpace\\c\\公交线路图\\ROUTES.txt","w");
                modify = Find_ROUTES_2(flag4+1,flag3);
                ROUTES[modify][2]=ROUTES[modify+1][2];
                ROUTES[modify][3]=distance;
                for(i=0;i<ROUTE_NUM;i++)
                {
                    fprintf(f3,"%d  %d  %d  %d\n",ROUTES[i][0],ROUTES[i][1],ROUTES[i][2],ROUTES[i][3]);
                }
                fclose(f3);
                delete_ROUTES(modify+1);
                load_data();        //把修改的文件再导入
            }
        }
        LoadMapDate();
        printf("\n删除成功！\n");
        break;
    default:
        printf("输入错误，请重新输入！\n");
        break;
    }
    load_data();
    LoadMapDate();
    return 0;
}
