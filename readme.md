my old code of kinda "engine"



#include <Windows.h>
#include <stdio.h>
#include <iostream>
#include <string>
#include <stdlib.h>
#include <sstream>
#include <cstring>
#include "sqlite3.h"
#include "gl/glew.h"
#include "gl/glut.h"

#include"sacred.h"


#pragma comment(lib, "glut32.lib")
#pragma comment(lib, "glew32.lib")


using namespace std;
// forvards

void findpointers(GOBJECT &obj);
string ftos(float value);
void create(string type,string name,int hierarhy,float damage,string location,float health,string actiontype,string actiontarget,int sector,string itemlist,string skillist,int maxitems,ITEMS* objItems,int maxskils,SKILS* objSkils,VERTEX pos,VERTEX target);

// global variables

LPDWORD threatID;
sqlite3 *db;
char *zErrMsg = 0;
int erc;

int pmax=0, skilmax=0, itemax=0;

int width, height;

GOBJECT * gobj=0;

GOBJECT tempgobj[108];

SKILS tempskils[108];
ITEMS tempitems[108];

ACTIVE_OBJ  act_gobj;




// structures



//static int callback(void *NotUsed, int argc, char **argv, char **azColName){
//  int i;
//  for(i=0; i<argc; i++){
//    printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
//  }
//  printf("\n");

//  return 0;
//}
string ftos(float value)
{
string str;
char chr[128];

sprintf(chr,"%f",value);
str=chr;

return str;
}//ftos


string itos(int value)
{
string str;
char chr[128];

sprintf(chr,"%d",value);
str=chr;

return str;
}//itos

static int nullcall(void *NotUsed, int argc, char **argv, char **azColName){return 0;}

static int initcall(void *NotUsed, int argc, char **argv, char **azColName){
  int i=0;
  //for(i=0; i<argc; i++){
    //printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
  //}
  printf("\n");

    tempgobj[pmax].status=1;

    for(int i=0;i<argc;i++)
    {
        string  astr;

        astr="name";
        if(azColName[i]==astr)tempgobj[pmax].name=argv[i];

        astr="type";
        if(azColName[i]==astr)tempgobj[pmax].type=argv[i];

        astr="location";
        if(azColName[i]==astr)tempgobj[pmax].location=argv[i];

        astr="actiontype";
        if(azColName[i]==astr)tempgobj[pmax].actiontype=argv[i];

        astr="actionTarget";
        if(azColName[i]==astr)tempgobj[pmax].actiontarget=argv[i];

        astr="itemlist";
        if(azColName[i]==astr)tempgobj[pmax].itemlist=argv[i];

        astr="skillist";
        if(azColName[i]==astr)tempgobj[pmax].skillist=argv[i];



        astr="posX";
        if(azColName[i]==astr)tempgobj[pmax].pos.x=atof(argv[i]);
        astr="posY";
        if(azColName[i]==astr)tempgobj[pmax].pos.y=atof(argv[i]);
        astr="posZ";
        if(azColName[i]==astr)tempgobj[pmax].pos.z=atof(argv[i]);


        astr="targetX";
        if(azColName[i]==astr)tempgobj[pmax].target.x=atof(argv[i]);
        astr="targetY";
        if(azColName[i]==astr)tempgobj[pmax].target.y=atof(argv[i]);
        astr="targetZ";
        if(azColName[i]==astr)tempgobj[pmax].target.z=atof(argv[i]);

        astr="damage";
        if(azColName[i]==astr)tempgobj[pmax].damage=atof(argv[i]);

        astr="health";
        if(azColName[i]==astr)tempgobj[pmax].health=atof(argv[i]);

        astr="sector";
        if(azColName[i]==astr)tempgobj[pmax].sector=atof(argv[i]);

        astr="hierarhy";
        if(azColName[i]==astr)tempgobj[pmax].hierarhy=atof(argv[i]);




    }

    pmax++;

  return 0;
}


static int skilcall(void *NotUsed, int argc, char **argv, char **azColName){

    for(int i=0;i<argc;i++)
    {
        string  astr;

        astr="listname";
        if(azColName[i]==astr)tempskils[skilmax].listname=argv[i];

        astr="skil";
        if(azColName[i]==astr)tempskils[skilmax].skil=argv[i];

        astr="value";
        if(azColName[i]==astr)tempskils[skilmax].value=atof(argv[i]);

    }

skilmax++;
return 0;
}//skilcall


static int itemcall(void *NotUsed, int argc, char **argv, char **azColName){

    for(int i=0;i<argc;i++)
    {
        string  astr;

        astr="listname";
        if(azColName[i]==astr)tempitems[itemax].listname=argv[i];

        astr="item";
        if(azColName[i]==astr)tempitems[itemax].item=argv[i];

        astr="status";
        if(azColName[i]==astr)tempitems[itemax].status=argv[i];

        astr="count";
        if(azColName[i]==astr)tempitems[itemax].value=atof(argv[i]);



    }

itemax++;
return 0;
}//skilcall


int sendsql( const char*string, sqlite3_callback cfunct )
{
    int rc;

    rc = sqlite3_exec(db, string, cfunct, 0, &zErrMsg);
    if( rc!=SQLITE_OK ){
        fprintf(stderr, "SQL error: %s\n", zErrMsg);
        sqlite3_free(zErrMsg);
    }



//typedef int (*sqlite3_callback)(void*,int,char**, char**);

return 0;
}//sendsql


int sqlinit()
{
    int rc;

    rc = sqlite3_open("prime.db", &db);

    if( rc ){
        fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        //exit(1);
    }

return 0;
}//sqlinit


void sqlstop()
{

    sqlite3_close(db);

}//sqlstop


ACTIVE_OBJ sort(ACTIVE_OBJ  sortobj)
{
    GOBJECT * tempobj;
    GOBJECT * tempobj2;
    int ti=0;

    tempobj=new GOBJECT[sortobj.count];

    for(int i=0;i<sortobj.count;i++)
    {

        if(sortobj.gobj[i].status !=0) { tempobj[ti]=sortobj.gobj[i]; ti++; }


    }

    tempobj2=new GOBJECT[ti];

    for(int i=0;i<ti;i++)
    {
        tempobj2[i]=tempobj[i];
    }

sortobj.gobj=tempobj2;
sortobj.count=ti;

return sortobj;

}//sort


ACTIVE_OBJ add(ACTIVE_OBJ  addobj,GOBJECT newgobject )
{
    int i=0;
    GOBJECT * tempobj;
    GOBJECT     tmp[10];

    tempobj=new GOBJECT[addobj.count+2];




for(i=0;i<addobj.count;i++)tempobj[i]=addobj.gobj[i];

tempobj[i]=newgobject;

//for(i=0;i<5;i++)tmp[i]=tempobj[i];


//tmp[0]=tmp[0];

addobj.gobj=tempobj;

addobj.count++;

return addobj;


}//add

void getskils(ACTIVE_OBJ &obj,int i)
{    int rc;
    skilmax=0;
    string listname=obj.gobj[i].name;
    string sqcom="select * from skillist where listname='"+listname+"'";
    rc=rc=sendsql(sqcom.c_str(),skilcall);

    tempskils[0]=tempskils[0];

obj.gobj[i].maxskils=skilmax;

obj.gobj[i].objSkils=new SKILS[skilmax];

for(int j=0;j<skilmax;j++)obj.gobj[i].objSkils[j]=tempskils[j];

tempskils[0]=tempskils[0];

}//getskils


void getitems(ACTIVE_OBJ &obj,int i )
{
    itemax=0;
    int rc;
    string listname=obj.gobj[i].name;
    string sqcom="select * from itemlist where listname='"+listname+"'";
    rc=rc=sendsql(sqcom.c_str(),itemcall);


    obj.gobj[i].maxitems=itemax;

    obj.gobj[i].objItems=new ITEMS[itemax];

    for(int j=0;j<itemax;j++)obj.gobj[i].objItems[j]=tempitems[j];



tempitems[0]=tempitems[0];
}

void sortitems(GOBJECT &obj)
{
    int rc;
    ITEMS *itemarray;
    ITEMS *temparray;

    itemarray=new ITEMS[obj.maxitems+1];

    int i=0;
    int t=0;
    for(i=0;i<obj.maxitems;i++)
    {
        if(obj.objItems[i].value !=0 )
        {
            itemarray[t]=obj.objItems[i];
            t++;
        }

    }

    temparray=new ITEMS[t];

    for(i=0;i<t;i++) temparray[i]=itemarray[i];


    obj.objItems=temparray;
    obj.maxitems=t;

    rc=sendsql("delete from itemlist where count=0",nullcall);

}//sortitems


void additem(GOBJECT &obj, string item, string status, int value)
{
    ITEMS newitem;

    bool newit=true;
    int rc;

    string c;

    for(int i=0;i<obj.maxitems;i++)
    {
        if(obj.objItems[i].item==item)
        {
            obj.objItems[i].value+=value;
            newit=false;
            // db update here
            char count[30];
            sprintf(count,"%d",obj.objItems[i].value);
            string val=count;
                c="update itemlist set count="+val+" where listname='"+obj.name+"' and item='"+item+"'";
                rc=sendsql(c.c_str(),nullcall);
        }

    }

    if(newit)
    {
    newitem.item=item;
    newitem.listname=obj.name;
    newitem.status=status;
    newitem.value=value;



    char vcc[30];

    sprintf(vcc,"%d",value);

    string vc=vcc;

    ITEMS *itemarray;

    itemarray=new ITEMS[obj.maxitems+2];

    int i=0;
    for(i=0;i<obj.maxitems;i++)itemarray[i]=obj.objItems[i];
    itemarray[i]=newitem;
    obj.objItems=itemarray;
    obj.maxitems++;

    //obj.actiontype="faking";

    c="delete from itemlist where status='"+status+"' and listname='"+obj.name+"' and item='"+item+"' and count="+vc;
    //rc=sendsql(c.c_str(),nullcall);

    string  c1="insert into itemlist ('status','listname','item','count')  values ('"+status+"','"+obj.name+"','"+item+"','"+vc+"')";
    //rc=sendsql(c1.c_str(),nullcall);
    }

    sortitems(obj);

    for(int i=0;i<act_gobj.count;i++)findpointers(act_gobj.gobj[i]);
}//additem


int  loadDB()
{
    int rc;


    pmax=0;
    rc=sendsql("select * from players",initcall);

    //tempgobj[0]=tempgobj[0];

    act_gobj.gobj=new GOBJECT[pmax];
    act_gobj.count=pmax;
    for(int i=0;i<pmax;i++)
    {
        act_gobj.gobj[i]=tempgobj[i];
    }



    act_gobj=sort(act_gobj);

    pmax=act_gobj.count;

    for(int i=0;i<act_gobj.count;i++)
    {
        printf("%d name=%s type=%s \n", i,act_gobj.gobj[i].name.c_str(),
                                            act_gobj.gobj[i].type.c_str()
                                            );
    }

    for(int i=0;i<act_gobj.count;i++)
    {
        getskils(act_gobj,i);
        getitems(act_gobj,i);
        findpointers(act_gobj.gobj[i]);



    }//


//additem(act_gobj.gobj[0],"tnt","cargo",3);

return 0;
}//loaddb



void printhelp()
{
printf("\n\n");
printf("***************UI KEYS********************\n");
printf("*a..............................select up*\n");
printf("*z............................select down*\n");
printf("*m................................move to*\n");
printf("*1...............................camera 1*\n");
printf("***************UI KEYS********************\n");
}//print help

int main(int argc, char **argv)
{
    int rc;
    printf("DEEP 6.3 \n\n\n");



    rc=sqlinit();
    rc=loadDB();

    width=1024, height=600;


    glutInit(&argc, argv);
    glutInitWindowSize( width, height);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_DEPTH | GLUT_RGB | GLUT_MULTISAMPLE );
    //glutInitDisplayString("double rgb~8 depth~24 samples=4");
    glutCreateWindow("sacred engine-DEEP 6 SERVER v3");

        glewInit();
        if (GLEW_ARB_vertex_shader && GLEW_ARB_fragment_shader)
            printf("Ready for GLSL\n");
        else {
            printf("Not totally ready for GLSL :( \n");
            exit(1);
        }

    glutDisplayFunc(display);
    glutMouseFunc(mouse);
    glutMotionFunc(motion);
    glutPassiveMotionFunc(motion1);
    glutIdleFunc(idle);
    glutKeyboardFunc(key);
    glutReshapeFunc(resize);


        // configure a simple controller


    init();

        SYSTEMTIME st;
         GetSystemTime(&st);
         printf("\n\n\n ..** SERVER TIME**.. \n Year:%d\nMonth:%d\nDate:%d\nHour:%d\nMin:%d\nSecond:% d\n" ,st.wYear,st.wMonth,st.wDay,st.wHour,st.wMinute,st.wSecond);



printhelp();




    HANDLE serwerThread=CreateThread(     NULL,
                                NULL,
                                server,
                                NULL,
                                 NULL,
                                 threatID);








glutMainLoop();

sqlstop();
return 0;
}//main






void moveto(GOBJECT &obj,float tx, float ty, float tz)
{

        obj.target.x=tx;
        obj.target.y=ty;
        obj.target.z=tz;

if(obj.actiontype !="cargo" && obj.actiontype !="instaled")
    {

        obj.actiontype="goto";

        int rc;

        char cx[30];char cy[30];char cz[30];
        string sx,sy,sz;

        sprintf(cx,"%f",tx); sprintf(cy,"%f",ty); sprintf(cz,"%f",tz);

        sx=cx; sy=cy; sz=cz;

        string c="update players set targetX="+sx+", targetY="+sy+", targetZ="+sz+" where name='"+obj.name+"'";
        rc=sendsql(c.c_str(),nullcall);
    }



}//moveto



void uninstal(GOBJECT &obj)
{
    //obj.location="temp";
    for(int i=0;i<act_gobj.count;i++) if( act_gobj.gobj[i].name==obj.location) for (int si=0;si < act_gobj.gobj[i].maxitems;si++)
    {
        if(act_gobj.gobj[i].objItems[si].item==obj.name)if(act_gobj.gobj[i].objItems[si].value>0)act_gobj.gobj[i].objItems[si].value--;

        char cval[60];
        sprintf(cval,"%d",act_gobj.gobj[i].objItems[si].value);
        string sval=cval;
        string c="update itemlist set count="+sval+" where listname='"+act_gobj.gobj[i].name+"' and item='"+act_gobj.gobj[i].objItems[si].item+"'";
        int rc=sendsql(c.c_str(),nullcall);


        sortitems(act_gobj.gobj[i]);
    }//find that record and delete


}//uninstal



void instal(GOBJECT &what, GOBJECT &were, string mode)
{
    uninstal(what);

        string c="update players set location='"+were.name+"' where name='"+what.name+"'";
        int rc=sendsql(c.c_str(),nullcall);


    c="update players set actiontype='"+mode+"' where name='"+what.name+"'";
    rc=sendsql(c.c_str(),nullcall);
    what.actiontype=mode;
//were.actiontype="2faking";

    additem(were,what.name,mode,1);







what.location=were.name;


}// instal

GOBJECT &findsub(GOBJECT &obj)
{

for(int i=0;i<obj.maxitems;i++)
{

    char  cstr[30];
    string str;

    sscanf(obj.objItems[i].item.c_str(),"%s",cstr );

    str=cstr;
    //printf("%s\n",cstr );

    if(str=="sub") for(int si=0;si<act_gobj.count;si++) if(act_gobj.gobj[si].name==obj.objItems[i].item) return act_gobj.gobj[si];


}


//return &sub;
}

void moveinstaledstuff(GOBJECT &obj)
{

for(int i=0;i<obj.maxitems;i++) if(obj.objItems[i].status=="instaled")
{

    for(int ai=0;ai <act_gobj.count;ai++) if (act_gobj.gobj[ai].name==obj.objItems[i].item)
    {
        act_gobj.gobj[ai].pos=obj.pos;
    }


}

}//moveinstaledstuff

void movecargostuff(GOBJECT &obj)
{

for(int i=0;i<obj.maxitems;i++) if(obj.objItems[i].status=="cargo")
{

    for(int ai=0;ai <act_gobj.count;ai++) if (act_gobj.gobj[ai].name==obj.objItems[i].item)
    {
        act_gobj.gobj[ai].pos=obj.pos;
    }


}

}//movecargostuff

void settarget(GOBJECT &obj,string target)
{

obj.actiontarget=target;

        string c="update players set actionTarget='"+target+"' where name='"+obj.name+"'";
        //int rc=sendsql(c.c_str(),nullcall);


}//settarget


GOBJECT &findpointer(string name)
{

for(int i=0;i<act_gobj.count;i++)
{

    if(act_gobj.gobj[i].name==name) return act_gobj.gobj[i];

}


}//findpointer


void findpointers(GOBJECT &obj)
{
    for(int i=0; i<obj.maxitems; i++)
    {

        obj.objItems[i].pointer=&findpointer(obj.objItems[i].item);

    }




}//findpointer

void create(string type,string name,int hierarhy,float damage,string location,float health,string actiontype,string actiontarget,int sector,string itemlist,string skillist,int maxitems,ITEMS* objItems,int maxskils,SKILS* objSkils,VERTEX pos,VERTEX target)
{
GOBJECT ng;

ng.actiontarget=actiontarget;
ng.actiontype=actiontype;
ng.damage=damage;
ng.health=health;
ng.hierarhy=hierarhy;
ng.itemlist=itemlist;
ng.location=location;
ng.maxitems=maxitems;
ng.maxskils=maxskils;
ng.name=name;
ng.objItems=objItems;
ng.objSkils=objSkils;
ng.pos=pos;
ng.target=target;
ng.sector=sector;
ng.skillist=skillist;
ng.status=1;
ng.type=type;

bool(valid)=true;

for(int i=0;i<act_gobj.count;i++)if(name==act_gobj.gobj[i].name)valid=false;


if(valid)
{
    act_gobj=add(act_gobj,ng);

    string  c="insert into players ('actionTarget','actiontype','damage','health','hierarhy','itemlist','skillist','location','name','posX','posY','posZ','targetX','targetY','targetZ','sector','type')  values ('"+actiontarget+"','"+actiontype+"','"+ftos(damage)+"','"+ftos(health)+"','"+ftos(hierarhy)+"','"+itemlist+"','"+skillist+"','"+location+"','"+name+"','"+ftos(pos.x)+"','"+ftos(pos.y)+"','"+ftos(pos.z)+"','"+ftos(target.x)+"','"+ftos(target.y)+"','"+ftos(target.z)+"','"+ftos(sector)+"','"+type+"')";
    //int rc=sendsql(c.c_str(),nullcall);

}




}//create



void createOre(string name,VERTEX orepos)
{
VERTEX oretarget;



oretarget.x=0;
oretarget.y=0;
oretarget.z=0;

ITEMS *oreitems=0;
SKILS *oreskils=0;

create("ore",
       name,
       2,
       0,
       "a0",
       1,
       "nop",
       "nop",
       666,
       "0",
       "ore_iron",
       0,
       oreitems,
       0,
       oreskils,
       orepos,
       oretarget);




}//

GOBJECT &findObject(string name)
{


    for(int i=0;i<act_gobj.count;i++) if(act_gobj.gobj[i].name==name)return act_gobj.gobj[i];



}//findobject

GOBJECT &findOreItem(GOBJECT &obj)
{

char ch[30];



for(int i=0;i<obj.maxitems;i++)
{
    sscanf(obj.objItems[i].item.c_str(),"%s",ch);

    string csh=ch;
    if(csh=="ore")
    {
        GOBJECT &ret=obj.objItems[i].pointer[0];
        return ret;
    }
}

}//findore


void gameloop()
{

        SYSTEMTIME st;
        char cstime[128];
         GetSystemTime(&st);
         sprintf(cstime,"%d%d%d%d%d%d" ,st.wYear,st.wMonth,st.wDay,st.wHour,st.wMinute,st.wSecond);
        string stime=cstime;



    //act_gobj.gobj[3].objItems[2]=act_gobj.gobj[3].objItems[2];

    for(int i=0;i<act_gobj.count;i++)
    {

        moveinstaledstuff(act_gobj.gobj[i]);
        movecargostuff(act_gobj.gobj[i]);

        if(act_gobj.gobj[i].type=="organism" && act_gobj.gobj[i].location=="base")
        {
            for(int ai=0;ai<act_gobj.gobj[i].maxitems;ai++) if(act_gobj.gobj[i].objItems[ai].item=="map")
            {
                //printf("map found\n");
                float orex,orey,orez;

                for(int mi=0;mi<act_gobj.count;mi++) if(act_gobj.gobj[mi].name=="map")
                {
                    for(int ski=0;ski<act_gobj.gobj[mi].maxskils;ski++)
                    {
                        if(act_gobj.gobj[mi].objSkils[ski].skil=="x")orex=act_gobj.gobj[mi].objSkils[ski].value;
                        if(act_gobj.gobj[mi].objSkils[ski].skil=="y")orey=act_gobj.gobj[mi].objSkils[ski].value;
                        if(act_gobj.gobj[mi].objSkils[ski].skil=="z")orez=act_gobj.gobj[mi].objSkils[ski].value;

                    }// read cords fromm map

                    //printf("ore cordinates x%f  y%f  z%f \n",orex,orey,orez);
                    moveto(act_gobj.gobj[i],orex,orey,orez);

                    GOBJECT &sub=findsub(act_gobj.gobj[i]);

                    instal(act_gobj.gobj[i], sub,"instaled" );

                    settarget(act_gobj.gobj[i],"ore");

                    for(int acti=0;acti<act_gobj.count;acti++) if(act_gobj.gobj[acti].name=="a0") instal(sub, act_gobj.gobj[acti],"free");

                }// find cordinates of ore


            }//search som map i items






        }// if ja meen and in base doin nothin


        if(act_gobj.gobj[i].type=="organism" )
        {
            if( act_gobj.gobj[i].actiontarget=="ore")
            {
                float di=dist(act_gobj.gobj[i].pos,act_gobj.gobj[i].target);

                if(di<20)
                {

                    string orename="ore "+act_gobj.gobj[i].name;
                    createOre(orename,act_gobj.gobj[i].pos);

                    //printf("ore found !!!\n");
                    GOBJECT &sub=findsub(act_gobj.gobj[i]);
                    sub.actiontarget="notarget";

                    //settarget(sub,"notarget");
                    act_gobj.gobj[i].actiontarget="base";


                    //sub.actiontype="FAK1";



                    instal(findObject(orename),sub,"cargo");

                    //act_gobj.gobj[3]=act_gobj.gobj[3];
                    VERTEX basepos=findObject("base").pos;
                    act_gobj.gobj[i].target=basepos;



                }// get ore
            }//if going for ore




            if( act_gobj.gobj[i].actiontarget=="base")
            {
                float di=dist(act_gobj.gobj[i].pos,act_gobj.gobj[i].target);

                if(di<20)
                {

                    //printf("base reach !!!\n");
                    GOBJECT &sub=findsub(act_gobj.gobj[i]);
                    sub.actiontarget="notarget";
                    //settarget(sub,"notarget");
                    act_gobj.gobj[i].actiontarget="ore";

                    //act_gobj.gobj[3]=act_gobj.gobj[3];

                    GOBJECT &oreincargo=findOreItem(sub);

                    GOBJECT &base=findObject("base");

                    additem(base,"ore base","cargo",1);
                    instal(oreincargo, base   ,"cargo");




                for(int it=0;it<act_gobj.gobj[i].maxitems;it++)
                {
                    gobject *map=act_gobj.gobj[i].objItems[it].pointer;



                            if(map[0].name=="map")
                            {
                                //printf("map found\n");
                                float orex,orey,orez;


                                for(int ski=0;ski<map[0].maxskils;ski++)
                                {
                                    if(map[0].objSkils[ski].skil=="x")orex=map[0].objSkils[ski].value;
                                    if(map[0].objSkils[ski].skil=="y")orey=map[0].objSkils[ski].value;
                                    if(map[0].objSkils[ski].skil=="z")orez=map[0].objSkils[ski].value;

                                }// read cords fromm map
                                        act_gobj.gobj[i].target.x=orex;
                                        act_gobj.gobj[i].target.y=orey;
                                        act_gobj.gobj[i].target.z=orez;



                            }



                }




                }// get ore
            }//going for ore





        }//if mining


        if(act_gobj.gobj[i].type=="wehicle" )
        {
            if(act_gobj.gobj[i].actiontarget=="notarget")
            {

                for(int it=0;it<act_gobj.gobj[i].maxitems;it++)
                {
                    act_gobj.gobj[0]=act_gobj.gobj[0];
                    gobject *navigator=act_gobj.gobj[i].objItems[it].pointer;

                    if(navigator[0].type=="organism")
                    {
                        act_gobj.gobj[i].actiontarget=navigator[0].actiontarget;
                        act_gobj.gobj[i].target=navigator[0].target;


                        act_gobj.gobj[i].actiontype="goto";
                    }//if organism

                }//for

            }//ifnotarget




        }// if ja som wehicle  and actualy doing nothing



    }//for aall pepel in da world
}//gameloop

**************************************engine



#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include"sacred.h"
#include"gl/glut.h"
#include <string>
#include "texture.h"

using namespace std;

//forvards
VERTEX GetOGLPos(int x, int y, float z);
VERTEX a3Dto2D(VERTEX cords);
VERTEX a2dto3d(int x, int y, float z);
float dist(VERTEX a, VERTEX b);
void getdelta();

//globals
float delta=0;
//Light position
float p_light[3] = {1.0,0.5,1.0};

//Light lookAt
float l_light[3] = {0,0,-5};

GLuint GridTexture, backTexture, spritetxt,subtxt;
Texture texture[6];

float marginup, margindown, marginleft, marginright;

int frame=0,time,timebase=0;
float fps;

int curentselection=0;

char sc[6];

float camx,camy, camoldx, camoldy;
float sensitivity=0.01f;
float camry=-160;
float camrx=-90;
float campx=0;
float campy=1;
float campz=0;

int mouseX,mouseY;

VERTEX mouset;

typedef struct                                        // Structure For 3D Points
{
    float   u,v;                                    // uv cords

}UV;

typedef struct                                        // Structure For 3D Points
{
float     nx,ny,nz;                                // normal
}NORMAL;

typedef    struct                                        // Structure For An poligon
{
  int        points[4];                                // kuri tris vertekshi satada poligonu
  int  material;
  int   normal[4];                                    // kuri tris normalji ir poligonam
} POLYGON;

typedef    struct                                        // Structure For An mesh
{
 int        numpoly;                                    // cik poligonu mesha
 int    *polys;                                // kuri poligoni satada meshu
} MESH;


typedef    struct                                        // Structure For An mesh
{
    int nummeshes;
    MESH * meshs;
    POLYGON * poly;
    VERTEX * vertice;
    UV *  uvs;
    NORMAL * norm;
                            // kuri poligoni satada meshu
} OBJECT;


VERTEX * vert=0;
POLYGON * poly=0;
NORMAL * norm=0;
UV* uves=0;

MESH * xMesh=0;
OBJECT xObject;

OBJECT cube,sphere,baal,sub,chuvak,torp,ore,base;

struct COLOR {   // Declare CELL bit field
    GLfloat r;
    GLfloat g;
    GLfloat b;
    GLfloat a;
} color;


COLOR* colorbuffer=0;

COLOR* grid=0;
COLOR* clear=0;


typedef enum {
   MODE_BITMAP,
   MODE_STROKE
} mode_type;

static mode_type mode;
static int font_index;


   void* bitmap_fonts[7] = {
      GLUT_BITMAP_9_BY_15,
      GLUT_BITMAP_8_BY_13,
      GLUT_BITMAP_TIMES_ROMAN_10,
      GLUT_BITMAP_TIMES_ROMAN_24,
      GLUT_BITMAP_HELVETICA_10,
      GLUT_BITMAP_HELVETICA_12,
      GLUT_BITMAP_HELVETICA_18
   };


float Target(VERTEX SHIP_Pos, VERTEX t_Pos)
{


    //D3DXVECTOR3 RELATIVE_POS=t_Pos-SHIP_Pos;

    //t_hip= sqrt(  (RELATIVE_POS[0]*RELATIVE_POS[0])+(RELATIVE_POS[1]*RELATIVE_POS[1])+(RELATIVE_POS[2]*RELATIVE_POS[2])    );// distance ir kvadratsakne no kordinatu starpibas paceltas kvadrata sumas d=sqrt  (x2-x1)^2+(y2-y1)^2+(z2-z1)^2

    //t_hip=distance3d( t_Pos, SHIP_Pos );

    float t_angle= -atan2(SHIP_Pos.x - t_Pos.x, SHIP_Pos.z - t_Pos.z) * 180 / PI;

    //t_angle2= atan2(SHIP_Pos[1] - t_Pos[1], distance(SHIP_Pos[0] , t_Pos[0],SHIP_Pos[2] , t_Pos[2]) ) * 180 / D3DX_PI;//  lenkjis starp punktiem atan2(x-x1,y-y1);

    return t_angle;

}

// Debug output to a text file 'output.txt', so we can dump our information out
// to a debug text file, so we can look at it
void dprintf(const char *fmt, ...)
{
    va_list parms;
    char buf[256] = {0};

    // Try to print in the allocated space.
    va_start(parms, fmt);
    vsprintf (buf, fmt, parms);
    va_end(parms);

    // Write the information out to a txt file
    FILE *fp = fopen("output.txt", "a+");
    fprintf(fp, "%s", buf);
    fclose(fp);
}// End dprintf(..)


void readstr(FILE *f,char *string)                            // Reads A String From File (f)
{
    do                                        // Do This
    {
        fgets(string, 255, f);                            // Gets A String Of 255 Chars Max From f (File)
    } while ((string[0] == '/') || (string[0] == '\n'));                // Read Again If Line Has Comment Or Is Blank
    return;                                        // Return
}





OBJECT loadx(const char*ofile )
{
    char    oneline[255];

    int filelenght=0;
    char    meshname[255];
    int numvertices=0;
    int numfaces=0;
    int numnormals=0;
    int numuv=0;

    int nummaterial=0;

    int matasign=0;

    int listnormals=0;

    int p1,p2,p3,p4;

    float x,y,z,u,v;




    printf(".",ofile);
    FILE* filein  = fopen(ofile, "rt");



    readstr(filein,oneline);                            // Jumps To Code That Reads One Line Of Text From The File
    sscanf(oneline, "xof %d\n", &filelenght);

    //printf( oneline );
    //printf("lenght %d \n",filelenght);


for(int i=0; i<filelenght-200;i++)
{
    readstr(filein,oneline);

sscanf(oneline,"    Mesh %s \n", &meshname);
if(strlen(meshname)<255)    {printf(".",meshname); i=filelenght+10;}

}

readstr(filein,oneline);
sscanf(oneline,"%d",&numvertices);
//printf( "\n\n vertices %d \n",numvertices );

vert=new VERTEX[numvertices];


for(int i=0; i<numvertices; i++)
{
    readstr(filein,oneline);
    sscanf(oneline, "%f %*s %f %*s %f \n",&x, &y, &z);

    vert[i].x=x;
    vert[i].y=y;
    vert[i].z=z;

    //printf("\n %d  x %f y %f z %f",i,x,y,z);


}

readstr(filein,oneline);
sscanf(oneline,"%d",&numfaces);
//printf( "\n\n faces %d \n",numfaces );

poly=new POLYGON[numfaces];

for(int i=0; i<numfaces; i++)
{

    readstr(filein,oneline);
    sscanf(oneline, "%d %*c %d %*c %d %*c %d\n",&p1, &p2, &p3, &p4);

    poly[i].points[0]=p2;
    poly[i].points[1]=p3;
    poly[i].points[2]=p4;
    poly[i].points[3]=p1;


    //printf("\n poly %d  vertices v1=%d v2=%d v3=%d count=%d ",i+1,p2,p3,p4,p1);

}

readstr(filein,oneline);

readstr(filein,oneline);
sscanf(oneline,"%d",&numnormals);
//printf( "\n\n normals %d \n",numnormals );



norm=new NORMAL[numnormals];

for(int i=0; i<numnormals; i++)
{
    readstr(filein,oneline);
    sscanf(oneline, "%f %*c %f %*c %f \n",&x, &y, &z);

    norm[i].nx=x;
    norm[i].ny=y;
    norm[i].nz=z;

    //printf("\n %d normal x %f y %f z %f",i,x,y,z);


}



readstr(filein,oneline);
sscanf(oneline,"%d",&listnormals);
//printf( "\n\n listnormals %d \n",listnormals );



for(int i=0; i<numfaces; i++)
{

    readstr(filein,oneline);
    sscanf(oneline, "%d %*c %d %*c %d %*c %d\n",&p1, &p2, &p3, &p4);


    poly[i].normal[0]=p2;
    poly[i].normal[1]=p3;
    poly[i].normal[2]=p4;
    poly[i].normal[3]=p1;


    //printf("\n face normals %d  normal n1=%d n2=%d n3=%d count=%d ",i+1,p2,p3,p4,p1);

}

readstr(filein,oneline);
readstr(filein,oneline);

readstr(filein,oneline);
sscanf(oneline,"%d",&numuv);
//printf( "\n\n numuv %d \n",numuv );

uves=new UV[numuv];

for(int i=0; i<numuv; i++)
{
    readstr(filein,oneline);
    sscanf(oneline, "%f %*c %f \n",&u, &v);

    vert[i].u=u;
    vert[i].v=v;

    uves[i].u=u;
    uves[i].v=u;


    //printf("\n %d vertex u=%f v=%f",i,u,v);

}

readstr(filein,oneline);
readstr(filein,oneline);


readstr(filein,oneline);
sscanf(oneline,"%d",&nummaterial);
//printf( "\n\n materials in object %d \n",nummaterial );



readstr(filein,oneline);
sscanf(oneline,"%d",&matasign);
//printf( "\n\n asigned materials on faces %d \n",matasign );


for(int i=0; i<matasign; i++)
{
    readstr(filein,oneline);
    sscanf(oneline, "%d\n",&p1);

    poly[i].material=p1;


    //printf("\n %d face with material id=%d ",i,p1);

}


xMesh=new MESH[1];

xMesh[0].polys=new int[numfaces];

xMesh[0].numpoly=numfaces;

for(int i=0; i<numfaces; i++)
{
    xMesh[0].polys[i]=i;
}


xObject.meshs=xMesh;
xObject.norm=norm;
xObject.nummeshes=1;
xObject.poly=poly;
xObject.uvs=uves;
xObject.vertice=vert;



return xObject;
}//load x

void drawmesh( OBJECT obj, int mode)
{




if(mode==1)
{

for(int nummeshes=0; nummeshes<obj.nummeshes; nummeshes++)// zimejam tik meshu cik objekat jabut
{
    int v, v1, v2;
    int uc,uc1,uc2;




    NORMAL n,n1,n2;
    for(int i=0; i<obj.meshs[nummeshes].numpoly  ; i++) // zimejam tik poligonu cik mesha rakstits
    {

            v=obj.poly[i].points[0];
            v1=obj.poly[i].points[1];
            v2=obj.poly[i].points[2];


            uc=obj.poly[i].normal[0];
            uc1=obj.poly[i].normal[1];
            uc2=obj.poly[i].normal[2];

            n=obj.norm[uc];
            n1=obj.norm[uc1];
            n2=obj.norm[uc2];


            glBegin(GL_TRIANGLES);

            glNormal3f(n.nx,n.ny,n.nz);glTexCoord2f(obj.vertice[uc].u, obj.vertice[uc].v);glVertex3f(obj.vertice[v].x, obj.vertice[v].y, obj.vertice[v].z);    // lower left vertex

            glNormal3f(n1.nx,n1.ny,n1.nz);glTexCoord2f(obj.vertice[uc1].u, obj.vertice[uc1].v);glVertex3f(obj.vertice[v1].x, obj.vertice[v1].y, obj.vertice[v1].z);    // lower right vertex

            glNormal3f(n2.nx,n2.ny,n2.nz);glTexCoord2f(obj.vertice[uc2].u, obj.vertice[uc2].v);glVertex3f(obj.vertice[v2].x, obj.vertice[v2].y, obj.vertice[v2].z);    // upper vertex


            glEnd();


    }// for polys




}//for meshes

}// if mode =1

}//drawmesh









bool LoadTGA(Texture *, char *);

GLuint LoadGLTextures(char * path)                                            // Load Bitmaps And Convert To Textures
{
    int Status=FALSE;
                                            // Status Indicator

    GLuint newtxt;

    // Load The Bitmap, Check For Errors.
    if (LoadTGA(&texture[0], path) )
    {
        Status=TRUE;                                            // Set The Status To TRUE


            // Typical Texture Generation Using Data From The TGA ( CHANGE )
            glGenTextures(1, &texture[0].texID);                // Create The Texture ( CHANGE )
            glBindTexture(GL_TEXTURE_2D, texture[0].texID);
            glTexImage2D(GL_TEXTURE_2D, 0, texture[0].bpp / 8, texture[0].width, texture[0].height, 0, texture[0].type, GL_UNSIGNED_BYTE, texture[0].imageData);
            glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
            glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);

            if (texture[0].imageData)                        // If Texture Image Exists ( CHANGE )
            {
                free(texture[0].imageData);                    // Free The Texture Image Memory ( CHANGE )
            }

    }

    newtxt=texture[0].texID;
    return newtxt;                                                // Return The Status
}
























void resetPerspectiveProjection() {
    glMatrixMode(GL_PROJECTION);
    glPopMatrix();
    glMatrixMode(GL_MODELVIEW);
}

void setOrthographicProjection() {


    int w=width; int h=height;

    // switch to projection mode
    glMatrixMode(GL_PROJECTION);
    // save previous matrix which contains the
    //settings for the perspective projection
    glPushMatrix();
    // reset matrix
    glLoadIdentity();
    // set a 2D orthographic projection
    gluOrtho2D(0, w, 0, h);
    // invert the y axis, down is positive
    glScalef(1, -1, 1);
    // mover the origin from the bottom left corner
    // to the upper left corner
    glTranslatef(0, -h, 0);
    glMatrixMode(GL_MODELVIEW);
}

void print_bitmap_string(void* font, char* s)
{


   if (s && strlen(s)) {
      while (*s) {
         glutBitmapCharacter(font, *s);
         s++;
      }
   }
}//print

void drawtext (int x, int y, int z, string text ) {

    char end={'\0'};
    char txt[128];

int i;

    if(    text.length()<128)
    {
        for( i=0;i<text.length();i++) txt[i]=text[i];
        txt[i]=end;






    setOrthographicProjection();
    glPushMatrix();
    glLoadIdentity();

         glRasterPos2f(x, y);
        if (z==0)        print_bitmap_string(bitmap_fonts[1], txt );

    glPopMatrix();
    resetPerspectiveProjection();

    }


}

GLuint EmptyTexture(int sizex, int sizey)                            // Create An Empty Texture
{
    GLuint txtnumber;                        // Texture ID
    unsigned int* data;                        // Stored Data

    // Create Storage Space For Texture Data (128x128x4)
    data = (unsigned int*)new GLuint[((sizex * sizey)* 4 * sizeof(unsigned int))];



    ZeroMemory(data,((sizex * sizex)* 4 * sizeof(unsigned int)));    // Clear Storage Memory

    glGenTextures(1, &txtnumber);                    // Create 1 Texture
    glBindTexture(GL_TEXTURE_2D, txtnumber);            // Bind The Texture
    glTexImage2D(GL_TEXTURE_2D, 0, 4, sizex, sizey, 0,
        GL_RGBA, GL_UNSIGNED_BYTE, data);            // Build Texture Using Information In data
    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);

    delete [] data;                            // Release data

    return txtnumber;                        // Return The Texture ID
}

void drawUI()
{


        SYSTEMTIME st;
        char cstime[128];
         GetSystemTime(&st);
         sprintf(cstime,"Year:%d Month:%d Date:%d Hour:%d Min:%d Second:% d " ,st.wYear,st.wMonth,st.wDay,st.wHour,st.wMinute,st.wSecond);
        string stime=cstime;

char position[128];
char tposition[128];
glColor3f(1.0, 1.0, 1.0);

drawtext(300,10,0,stime);

glColor3f(0.5, 0.5, 1.0);

for(int i=0;i<act_gobj.count;i++)
{

    if (act_gobj.gobj[i].actiontype !="instaled" && act_gobj.gobj[i].actiontype !="cargo"){
glPushMatrix();
VERTEX p,t;

    p.x=act_gobj.gobj[i].pos.x; p.y=act_gobj.gobj[i].pos.y; p.z=act_gobj.gobj[i].pos.z;
    t.x=act_gobj.gobj[i].target.x; t.y=act_gobj.gobj[i].target.y; t.z=act_gobj.gobj[i].target.z;

VERTEX textpos= a3Dto2D(p);
drawtext (textpos.x,-textpos.y+height,textpos.z,act_gobj.gobj[i].name+" status:"+act_gobj.gobj[i].actiontype+" for "+act_gobj.gobj[i].actiontarget+" in "+act_gobj.gobj[i].location);




sprintf(position,"px:%4.2f   py:%4.2f  pz:%4.2f",act_gobj.gobj[i].pos.x,act_gobj.gobj[i].pos.y,act_gobj.gobj[i].pos.z);

drawtext (textpos.x,-textpos.y+height+10,textpos.z,position);



sprintf(tposition,"tx:%4.2f   ty:%4.2f  tz:%4.2f",act_gobj.gobj[i].target.x,act_gobj.gobj[i].target.y,act_gobj.gobj[i].target.z);

drawtext (textpos.x,-textpos.y+height+20,textpos.z,tposition);

char cdist[128];
sprintf(cdist,"dist:%4.2f",dist(p,t));
drawtext (textpos.x,-textpos.y+height+30,textpos.z,cdist);

glPopMatrix();
    }//if
}//for


int n=curentselection;
int step=10;






sprintf(position,"px:%4.2f   py:%4.2f  pz:%4.2f",act_gobj.gobj[n].pos.x,act_gobj.gobj[n].pos.y,act_gobj.gobj[n].pos.z);





sprintf(tposition,"tx:%4.2f   ty:%4.2f  tz:%4.2f",act_gobj.gobj[n].target.x,act_gobj.gobj[n].target.y,act_gobj.gobj[n].target.z);





drawtext (6,42,0,act_gobj.gobj[n].name+":Itemlist              "+position+"---"+tposition+" target "+act_gobj.gobj[n].actiontarget);
for(int i=0;i<act_gobj.gobj[n].maxitems;i++)
{

char number[128];
char value[128];
sprintf(number,"%d",i);
sprintf(value,"%d",act_gobj.gobj[n].objItems[i].value);

string nb=number;
string vb=value;
drawtext (6,60+(step*i),0,nb+":"+act_gobj.gobj[n].objItems[i].item);
drawtext (120,60+(step*i),0,":"+vb);



}



}//drawUI

void drawscene()
{
for(int i=0;i<act_gobj.count;i++)
{
glEnable(GL_DEPTH_TEST);

    VERTEX p,t;

    p.x=act_gobj.gobj[i].pos.x; p.y=act_gobj.gobj[i].pos.y; p.z=act_gobj.gobj[i].pos.z;
    t.x=act_gobj.gobj[i].target.x; t.y=act_gobj.gobj[i].target.y; t.z=act_gobj.gobj[i].target.z;

    glPushMatrix();
    //glTranslatef(0,0,-10);
    glTranslatef(p.x,p.y,p.z);
    glScalef(1,1,1);

    float ry=360+Target(p,t);

    glRotatef(-ry,  0,1,0);
    glEnable(GL_TEXTURE_2D);
    glEnable(GL_LIGHTING);
    glBindTexture(GL_TEXTURE_2D, texture[4].texID);
    glDisable(GL_BLEND);

    if(act_gobj.gobj[i].type=="organism")drawmesh( chuvak,1 );
    if(act_gobj.gobj[i].type=="wehicle")drawmesh( sub,1 );
    if(act_gobj.gobj[i].type=="torp100")drawmesh( torp,1 );
    if(act_gobj.gobj[i].type=="orefund")drawmesh( ore,1 );
    if(act_gobj.gobj[i].type=="base")drawmesh( base,1 );
glPopMatrix();

mouset=a2dto3d(mouseX,mouseY,0);

}

}//draw

void RenderToTexture( GLuint BlurTexture )                            // Renders To A Texture
{
    glViewport(0,0,width,height);                    // Set Our Viewport (Match Texture Size)

    drawscene();                            // Render The Helix

    glBindTexture(GL_TEXTURE_2D,BlurTexture);            // Bind To The Blur Texture

    // Copy Our ViewPort To The Blur Texture (From 0,0 To 128,128... No Border)
    glCopyTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, 0, 0, width, height, 0);

    glClearColor(0.0f, 0.0f, 0.05f, 0.5);                // Set The Clear Color To Medium Blue
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);        // Clear The Screen And Depth Buffer

    glViewport(0 , 0,width ,height);                    // Set Viewport (0,0 to 640x480)
}


void camlook(float px, float py, float pz, float rx, float ry, float rz)
{
float dist=1;




gluLookAt(px+cos(DEG2RAD(ry))*dist,py+cos(DEG2RAD(rx))*dist, pz+sin(DEG2RAD(ry))*dist,   px,py,pz,  0,1,0);

}//camlook


//=============================
void display()
{

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glColor4f(1.0, 1.0, 1.0, 0.5);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    //glBlendFunc(GL_SRC_ALPHA, GL_ONE);
    //glDisable(GL_BLEND);
    //glEnable(GL_BLEND);
    glLoadIdentity();



    camry+=camx*sensitivity;
    camrx+=camy*sensitivity;
    camlook(campx,campy,campz,   camrx,-camry,0);





RenderToTexture(  backTexture )    ;




        glDisable(GL_DEPTH_TEST);
        glDisable(GL_LIGHTING);
        glEnable(GL_TEXTURE_2D);
        glDisable(GL_BLEND);
glPushMatrix();
glLoadIdentity();

//void ViewOrtho();
//glBindTexture(GL_TEXTURE_2D, GridTexture);
//glBindTexture(GL_TEXTURE_2D, backTexture);
glBindTexture(GL_TEXTURE_2D, backTexture);

//glBindTexture(GL_TEXTURE_2D, depthTextureId);

glBegin(GL_QUADS);
        glTexCoord2f(0.0f, 0.0f);
        glVertex3f(marginleft, margindown,  -1.0f);    // Bottom Left Of The Texture and Quad

        glTexCoord2f(1.0f, 0.0f);
        glVertex3f( marginright, margindown,  -1.0f);    // Bottom Right Of The Texture and Quad

        glTexCoord2f(1.0f, 1.0f);
        glVertex3f(marginright,  marginup,  -1.0f);    // Top Right Of The Texture and Quad

        glTexCoord2f(0.0f, 1.0f);
        glVertex3f(marginleft,  marginup,  -1.0f);    // Top Left Of The Texture and Quad

        // Back Face
glEnd();




glBindTexture(GL_TEXTURE_2D, GridTexture);
glEnable(GL_BLEND);
glBegin(GL_QUADS);
        // Front Face

        glTexCoord2f(0.0f, 0.0f);
        glVertex3f(marginleft, margindown,  -1.0f);    // Bottom Left Of The Texture and Quad

        glTexCoord2f(1.0f, 0.0f);
        glVertex3f( marginright, margindown,  -1.0f);    // Bottom Right Of The Texture and Quad

        glTexCoord2f(1.0f, 1.0f);
        glVertex3f(marginright,  marginup,  -1.0f);    // Top Right Of The Texture and Quad

        glTexCoord2f(0.0f, 1.0f);
        glVertex3f(marginleft,  marginup,  -1.0f);    // Top Left Of The Texture and Quad

        // Back Face
glEnd();
glPopMatrix();



        glDisable(GL_LIGHTING);
        glDisable(GL_TEXTURE_2D);
        glColor3f(1.0, 1.0, 1.0);

    drawtext(1,10,0, "SACRED v 1.A" );

    frame++;
    time=glutGet(GLUT_ELAPSED_TIME);
    if (time - timebase > 1000) {
        fps=frame*1000.0/(time-timebase);
        sprintf(sc,"FPS:%4.2f",fps);
        timebase = time;
        frame = 0;

    }

    string fpss=sc;
    drawtext(width-90,10,0, fpss );





drawUI();







    glutSwapBuffers();
    getdelta();
}


//=======================
void mouse(int button, int state, int x, int y){}
void motion(int mx, int my)
{

    camx=camoldx-mx;
    camy=camoldy-my;
    //printf("mx=%f \n",camx);

}


void motion1(int mx, int my)
{


    camoldx = mx;
    camoldy = my;
    camx=0;
    camy=0;

    mouseX=mx;
    mouseY=my;

    //printf("mouse x%d mouse y%d   ---  mx%f   my%f   mz%f  \n",mouseX,mouseY,  mouset.x,mouset.y,mouset.z);
}

float oldtime;
SYSTEMTIME dst;
void getdelta()
{


    GetSystemTime(&dst);

    float curent=(dst.wSecond*1000)+dst.wMilliseconds;

    if(curent-oldtime>0)delta=(curent-oldtime)/1000;



    oldtime=curent;


}//getdelta


void idle()
{

    float speed;






    for(int i=0;i<act_gobj.count;i++)
    {
        VERTEX p,t;

            speed=0.01f;

            if(fps>6)
            {
                //printf("%f\n",delta);
                speed=delta*2;
            }

            if(act_gobj.gobj[i].type=="wehicle") speed*=6;


        p=act_gobj.gobj[i].pos;
        t=act_gobj.gobj[i].target;


        if(act_gobj.gobj[i].actiontype=="goto" && act_gobj.gobj[i].ac<1.00000f)  // then move
        {
            act_gobj.gobj[i].ac+=0.01f;
        }

        if(act_gobj.gobj[i].actiontype=="stop" && act_gobj.gobj[i].ac>0.00000f) act_gobj.gobj[i].ac-=0.01f;

        if(dist(p,t)<10 )act_gobj.gobj[i].actiontype="stop";



            float alpha=Target(t,p);
            //if(act_gobj.gobj[i].type=="wehicle")printf("alpha %f\n",alpha);
            act_gobj.gobj[i].pos.x+=cos(alpha)*speed*act_gobj.gobj[i].ac;
            act_gobj.gobj[i].pos.z+=sin(alpha)*speed*act_gobj.gobj[i].ac;






    }


    gameloop();


    glutPostRedisplay();
}
void key(unsigned char k, int x, int y)

{

    switch(k)
    {

        case 'a':
            if( curentselection < act_gobj.count-1)  curentselection++;
        break;

        case 'z':
            if( curentselection > 0)  curentselection--;
        break;

        case 'm':
            moveto(act_gobj.gobj[curentselection],mouset.x,0,mouset.z);

        break;

        case '1':
            campx=act_gobj.gobj[curentselection].pos.x;
            campz=act_gobj.gobj[curentselection].pos.z;
            campy=15;

        break;

    }


}//key





VERTEX a3Dto2D(VERTEX cords)
{
    GLint viewport[4];
    GLdouble modelview[16];
    GLdouble projection[16];
    GLfloat winX, winY, winZ;
    GLdouble posX, posY, posZ;

    glGetDoublev( GL_MODELVIEW_MATRIX, modelview );
    glGetDoublev( GL_PROJECTION_MATRIX, projection );
    glGetIntegerv( GL_VIEWPORT, viewport );

    winX = cords.x;
    winY = cords.y;
    winZ = cords.z;


    gluProject( winX, winY, winZ, modelview, projection, viewport, &posX, &posY, &posZ);

    VERTEX out;

    out.x=posX;
    out.y=posY;
    out.z=posZ;

    return out;
}

VERTEX GetOGLPos(int x, int y, float z)
{
    GLint viewport[4];
    GLdouble modelview[16];
    GLdouble projection[16];
    GLfloat winX, winY, winZ;
    GLdouble posX, posY, posZ;

    glGetDoublev( GL_MODELVIEW_MATRIX, modelview );
    glGetDoublev( GL_PROJECTION_MATRIX, projection );
    glGetIntegerv( GL_VIEWPORT, viewport );

    winX = (float)x;
    winY = (float)viewport[3] - (float)y;
    winZ=z;

    //glReadPixels( x, int(winY), 1, 1, GL_DEPTH_COMPONENT, GL_FLOAT, &winZ );

    gluUnProject( winX, winY, winZ, modelview, projection, viewport, &posX, &posY, &posZ);

    VERTEX out;

    out.x=posX;
    out.y=posY;
    out.z=posZ;

    return out;
}

float dist(VERTEX a, VERTEX b)
{

    float d=sqrt(  (a.x-b.x)*(a.x-b.x)   +    (a.y-b.y)*(a.y-b.y)   +   (a.z-b.z)*(a.z-b.z)    );

return d;

}//dist

VERTEX a2dto3d(int x, int y, float z)
{
    GLint viewport[4];
    GLdouble modelview[16];
    GLdouble projection[16];
    GLfloat winX, winY, winZ;
    GLdouble posX, posY, posZ;

    glGetDoublev( GL_MODELVIEW_MATRIX, modelview );
    glGetDoublev( GL_PROJECTION_MATRIX, projection );
    glGetIntegerv( GL_VIEWPORT, viewport );

    winX = (float)x;
    winY = (float)viewport[3] - (float)y;
    //winZ=z;

    glReadPixels( x, int(winY), 1, 1, GL_DEPTH_COMPONENT, GL_FLOAT, &winZ );

    gluUnProject( winX, winY, winZ, modelview, projection, viewport, &posX, &posY, &posZ);

    VERTEX out;

    out.x=posX;
    out.y=posY;
    out.z=posZ;

    return out;
}



void resize(int w, int h)
{

if (h == 0) h = 1;

   // ui.reshape( w, h);

    //ww=w;

    //wh=h;

    glViewport(0, 0, w, h);

    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();

    gluPerspective(60.0, (GLfloat)w/(GLfloat)h, 0.1, 10000.0);

    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();

   // manipulator.reshape(w, h);

    width = w;
    height = h;

int wsize=w*h;

    //delete[ ] colorbuffer;
    colorbuffer=new COLOR[wsize];

    grid=new COLOR[wsize];


COLOR black={1.0, 0.01, 0.01, 1.0};



COLOR c={0.5, 0.5, 0.5, 1.0};
for(int y=1; y<height;y+=10) {c.a=0.3+(1/y); for(int x=0;x<width;x++) grid[ x+(y*width) ]=c;}
COLOR c1={0.5, 0.5, 0.5, 1.0};
for(int y=1; y<height;y++)    {c1.a=0.3+(1/y); for(int x=0;x<width;x+=10) grid[ x+(y*width) ]=c1;}




//for(int i=0; i<wsize; i++)grid[i]=black;



            glGenTextures(1, &GridTexture);
            glBindTexture(GL_TEXTURE_2D, GridTexture);
    // Generate The Texture
        glTexImage2D(GL_TEXTURE_2D, 0, 4, width, height, 0, GL_RGBA, GL_FLOAT, grid);


            glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
            glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);




            GridTexture=GridTexture;

            backTexture= EmptyTexture(width,height);


            float z=0;

float factor=10;

            marginup=GetOGLPos( 0,0,z).y*factor;
            margindown=GetOGLPos( 0,height,z).y*factor;

            marginleft=GetOGLPos( 0,0,z).x*factor;
            marginright=GetOGLPos( width,0,z).x*factor;

}


void init()
{
    //glEnable(GL_TEXTURE_2D);                                    // Enable Texture Mapping ( NEW )
    glShadeModel(GL_SMOOTH);                                    // Enable Smooth Shading
    glClearColor(0.0f, 0.0f, 0.0f, 0.5f);                        // Black Background
    glClearDepth(1.0f);                                            // Depth Buffer Setup
    glEnable(GL_DEPTH_TEST);                                    // Enables Depth Testing
    //glDepthFunc(GL_LEQUAL);                                        // The Type Of Depth Testing To Do
    //glHint(GL_PERSPECTIVE_CORRECTION_HINT, GL_NICEST);            // Really Nice Perspective Calculations


    GLfloat LightAmbient[]= { 0.5f, 0.5f, 0.5f, 1.0f };
    GLfloat LightDiffuse[]= { 1.0f, 1.0f, 1.0f, 1.0f };

    GLfloat LightPosition[]= { 0.0f, 0.0f, 2.0f, 1.0f };

glLightfv(GL_LIGHT1, GL_AMBIENT, LightAmbient);

glLightfv(GL_LIGHT1, GL_DIFFUSE, LightDiffuse);

glLightfv(GL_LIGHT1, GL_POSITION,LightPosition);

glEnable(GL_LIGHT1);


    spritetxt=LoadGLTextures("Data/sprite.tga");
    subtxt=LoadGLTextures("Data/sub.tga");

    chuvak=loadx("chuvak.x");
    sub=loadx("sub.x");
    torp=loadx("torp.x");
    ore=loadx("ore.x");
    base=loadx("base.x");




}//
