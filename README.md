# -
试验田管理项目
#include<stdio.h>
#include<math.h>
#include<stdlib.h>
#include<time.h>
#include<string.h>
// 传感器数据记录（使用链表管理） 
typedef struct SensorRecord {     
	int field_id;          // 对应的田块 ID 
    char timestamp[20];    // 记录时间，格式"2025-04-10 14:30"     
	float temperature;     // 温度值，单位℃     
	float humidity;         // 湿度值，单位%     
	int is_abnormal;       // 是否异常：0-正常，1-异常     
	struct SensorRecord *next;  // 指向下一条记录 
} SensorRecord; 

struct SensorRecord *p=NULL;

//系统告警信息 
typedef struct Alert { 
    int field_id;          // 发生告警的田块 ID     
	char alert_time[20];   // 告警时间     
	char alert_type[30];   // 告警类型，如"高温告警"     
	char description[100]; // 告警详细描述 
    int severity;          // 严重程度：1-轻微，2-中等，3-严重     
	struct Alert *next;    // 指向下一条告警 
} Alert;
struct Alert*q=NULL; 

//试验田块信息 
typedef struct Field {
    int id;                 // 田块 ID，从 1 开始自动分配     
	char name[30];         // 田块名称，如"东区实验田 1 号"     
	char manager[20];      // 负责人姓名     
	int sensor_count;      // 该田块已安装的传感器数量 
	SensorRecord *head;	//让每个田块管理自己的链表 
	SensorRecord *tail;
	Alert *head1;
	Alert *tail1; 
} Field; 

//检测系统主机构 
typedef struct MonitorSystem { 
    Field **fields;        // 田块指针数组（二重指针）     
	int field_count;       // 当前田块数量     
	int field_capacity;    // 数组容量，初始为 10 
    SensorRecord *record_head;  // 传感器记录链表头指针   
	Alert *alert_head;     // 告警信息链表头指针     //后续废弃使用 
	int next_field_id;     // 下一个可用的田块 ID，用于保证每个新创建的田块都有一个唯一且连续的 ID。 
} MonitorSystem; 

//系统管理函数 
MonitorSystem*create_monitor_system(void){
	MonitorSystem *sys=(MonitorSystem*)malloc(sizeof(MonitorSystem));
	sys->field_capacity=10;
	sys->fields=(Field**)malloc(sys->field_capacity*sizeof(Field*));
	int i=0;
	for(i=0;i<sys->field_capacity;i++){
		sys->fields[i]=(Field*)malloc(sizeof(Field));
		sys->fields[i]->head =NULL;
		sys->fields[i]->tail =NULL;
		sys->fields[i]->sensor_count=0;
	} 
	sys->alert_head=NULL;
	sys->field_count=1;
	sys->next_field_id=1;
	sys->record_head=NULL;
	if(sys!=NULL) return sys;
	//出现问题：molloc函数 must be used in 运行时，不能作为全局变量 
	else {
	printf("分配失败！");
	return NULL;}
} 
void destroy_monitor_system(MonitorSystem *sys){//释放内存 
	 if (sys == NULL) return;
	 int i=0;
	 for(i=0;i<sys->field_capacity ;i++){
	 	if (sys->fields[i] != NULL) {
            Alert *temp=NULL;
            Alert *p=sys->fields[i]->head1;
            while(p!=NULL){
            	temp=p;
            	p=p->next;
            	free(temp);
			}
			SensorRecord *temp1;
			SensorRecord *p1;
			 while(p1!=NULL){
            	temp1=p1;
            	p1=p1->next;
            	free(temp1);
			}
            free(sys->fields[i]);
	 }
	free(sys->fields);
	free(sys);
	printf("系统资源已释放，退出成功！\n");
	
} }
  
void display_main_menu (void) {//显示系统主菜单
	printf("========================\n"); 
	printf("校园试验田环境监测系统\n");
	printf("========================\n");
	printf("1.试验田管理\n2.传感器数据管理\n3.数据查询与统计\n4.告警管理\n5.系统信息\n0.退出系统\n") ;
	printf("========================\n");
	printf("请选择操作（0-5）："); 
}
void expand_field_array(MonitorSystem*sys){//动态扩容田块数组 
	int new_capacity=sys->field_capacity*2;
	sys->fields=(Field**)realloc(sys->fields,new_capacity*sizeof(Field*));
}
//试验田管理函数

int add_field(MonitorSystem *sys,const char*manager){//添加田块 
	if(sys->field_count>=sys->field_capacity){
		expand_field_array(sys);
		if(sys->field_count>=sys->field_capacity){
			printf("扩容失败！"); 
			return 0; 
		}
	}
	Field *news=sys->fields[sys->field_count]; 
	if(news==NULL){
		printf("内存分配失败");
	return 0;}
	else{
	news->id=sys->next_field_id++;
	printf("请输入田块名称");
	scanf("%s",news->name);
	strcpy(news->manager ,manager);
	news->manager[sizeof(news->manager)-1] = '\0';
	news->sensor_count =0;
	news->head1=NULL;
	news->head=NULL;
	news->tail=NULL;
	news->tail1=NULL;
	sys->fields[sys->field_count]=news;//让数组的【1】对应为id为1 
	sys->field_count++; 
	printf("田块【%s】添加成功！ID：%d\n", news->name, news->id);
	return 1;}

} 

void display_all_fields(MonitorSystem*sys){//显示所有田块 
	int i;
	if(sys->field_count==0){
		printf("Not found!\n");
	}
	else{
	printf("=============================\n");
	printf("       所有试验田块共%d个    \n ",sys->field_count-1);
	printf("=============================\n");
	printf("ID\t名称\t负责人\t传感器数\n");
	printf("------------------------------\n");
	for(i=1;i<sys->field_count;i++){
		printf("%d\t",sys->fields[i]->id);
		printf("%s\t",sys->fields[i]->name);
		printf("%s\t",sys->fields[i]->manager );
		printf("%d",sys->fields[i]->sensor_count );
		printf("\n");
	}
	printf("=============================\n");
	}
}

Field* find_field_by_id(MonitorSystem*sys,int field_id){//按id找田块 
	Field* found=NULL;
	int i=0;
	for(i=1;i<sys->field_count;i++){
		if(sys->fields[i]->id==field_id){
			found=sys->fields[i];
			printf("ID=%d   ",found->id);
			printf("Name=%s   ",found->name);
			printf("manager=%s   ",found->manager );
			printf("sensor_count=%d\n",found->sensor_count);
			break;}
	}
	return found ;	
} 
int delete_field(MonitorSystem*sys,int field_id){//删除指定田块
	if (sys == NULL || field_id < 1 || field_id >= sys->field_count) {
        printf("删除失败：田块ID不合法！\n");
        return 0;
    }
	int i=1;
	printf("正在删除田块：%s(ID:%d)",sys->fields[field_id]->name ,field_id);
	for(i=sys->fields[field_id]->id;i<sys->field_count ;i++){
		 sys->fields[i] = sys->fields[i + 1];
	}
	sys->field_count--;
	printf("田块删除成功！\n",field_id);
	return 1;
}
void search_fields_by_name(MonitorSystem *sys, const char *keyword){//按名称查找田块 
	Field* found=NULL;
	int i=0;
	for(i=0;i<sys->field_count;i++){
		if(strcmp(sys->fields[i]->name,keyword)==0){//字符串比较用strcmp,若用==比较的是地址 
			found=sys->fields[i];
			printf("ID=%d   ",found->id);
			printf("Name=%s   ",found->name);
			printf("manager=%s   ",found->manager );
			printf("sensor_count=%d\n",found->sensor_count);
			break;}	
}} 
void search_fields_by_manager(MonitorSystem *sys, const char *manager_name){
		Field* found=NULL;
	int i=0;
	for(i=0;i<sys->field_count;i++){
		if(strcmp(sys->fields[i]->manager,manager_name)==0){//字符串比较用strcmp,若用==比较的是地址 
			found=sys->fields[i];
			printf("ID=%d   ",found->id);
			printf("Name=%s   ",found->name);
			printf("manager=%s   ",found->manager );
			printf("sensor_count=%d\n",found->sensor_count);
			break;}	
}
} 
//传感器数据管理
//auxiliury function to find the local time
void getcurrenttime(char* timestamp){
	time_t now=time(NULL);
	struct tm*timer=localtime(&now);
	//year
	timestamp[0]=(timer->tm_year+1900)/1000+'0';
	timestamp[1]=(timer->tm_year+1900)/100%10+'0';
	timestamp[2]=(timer->tm_year+1900)/10%100+'0';
	timestamp[3]=(timer->tm_year+1900)%10+'0';
	timestamp[4]='-';
	timestamp[5]=(timer->tm_mon+1)/10+'0';
	timestamp[6]=(timer->tm_mon+1)%10+'0';
	timestamp[7]='-';
	timestamp[8]=(timer->tm_mday)/10+'0';
	timestamp[9]=(timer->tm_mday)%10+'0';
	timestamp[10]=' ';
	timestamp[11]=(timer->tm_hour)/10+'0';
	timestamp[12]=(timer->tm_hour)%10+'0';
	timestamp[13]=':';
	timestamp[14]=timer->tm_min/10+'0';
	timestamp[15]=timer->tm_min%10+'0';
	timestamp[16]=':';
	timestamp[17]=timer->tm_sec/10+'0';
	timestamp[18]=timer->tm_sec%10+'0';
	timestamp[19]='\0';
	}
//add record of sensor
 int add_sensor_record(MonitorSystem *sys, int field_id, float temperature, float humidity){
	p=(SensorRecord*)malloc(sizeof(SensorRecord));
	if(p==NULL){
		printf("内存分配失败！"); 
		return;
	}
	else{
	p->field_id =field_id;
	p->temperature =temperature;
	p->humidity =humidity;
	p->next =NULL;
	p->is_abnormal=(p->temperature <10||p->temperature >35||p->humidity >85||p->humidity <40)?1:0;//异常为1  
	getcurrenttime(p->timestamp);
	if(sys->fields[field_id]->sensor_count==0){
 		sys->fields[field_id]->head=p;
 		sys->fields[field_id]->tail=p;
	 }
	 else{
		sys->fields[field_id]->tail->next=p;      
		sys->fields[field_id]->tail=p;  
	 }	
	sys->fields[field_id]->sensor_count++;
	if(sys->record_head==NULL){
		sys->record_head=p;
	}
	printf("ID为%d的传感器数据添加成功！\n",p->field_id);}
	return 0;}

void display_field_all_records(MonitorSystem *sys, int field_id){//显示所有记录
	if (sys == NULL) return;
	int i;
	if(sys->fields[field_id]->sensor_count==0){
		printf("Not found!\n");
		return;
	}
	SensorRecord *pq=sys->fields[field_id]->head;
	printf("======================================\n");
	printf("        该试验田块共%d个传感器       \n ",sys->fields[field_id]->sensor_count );
	printf("======================================\n");
	printf("ID\t温度\t湿度\t是否异常\t记录时间\n");//一开始用的空格来弄，发现无法对齐 
	printf("---------------------------------------\n");
	while (pq!= NULL){
		printf("%d\t",field_id);
		printf("%.2f℃\t",pq->temperature);
		printf("%.2f% \t",pq->humidity);
		if(pq->is_abnormal==1){	printf("1-异常\t",pq->is_abnormal);
		}        
		else{
			printf("0-正常\t",pq->is_abnormal);
		} 
		printf("%s\n",pq->timestamp);
		pq=pq->next;
	} 
	printf("=====================================\n");
	}
	
void display_latest_records(MonitorSystem *sys, int field_id, int count){//可以尝试用双向链表
	if (sys == NULL) return;
    if (sys->fields[field_id]== NULL ||sys->fields[field_id]->sensor_count == 0) {
        printf("该田块无传感器记录！\n");
        return;
    }
	int i=1;
	if(sys->fields[field_id]->sensor_count<count){
		printf("田块无满足数量的传感器！已为您显示全部记录！\n");
		display_field_all_records(sys, field_id); 
	} 
	else{
	SensorRecord *p = sys->fields[field_id]->head;
    SensorRecord *q = sys->fields [field_id]->head;
    for (i = 0; i < count; i++) {
        if (q == NULL) break;
        q = q->next;
    }
    while (q != NULL) {
        p = p->next;
        q = q->next;
    }
    // 输出从p开始的记录
    printf("======================================\n");
    printf("  田块【%s】最新%d条记录\n", sys->fields [field_id]->name, count);
    printf("======================================\n");
    printf("田块ID\t温度\t湿度\t是否异常\t记录时间\n");
    printf("---------------------------------------\n");
    while (p != NULL) {
        printf("%d\t%.2f℃\t%.2f%%\t%s\t\t%s\n",
               p->field_id, p->temperature, p->humidity,
               p->is_abnormal ? "1-异常" : "0-正常", p->timestamp);
        p = p->next;
    }
    printf("======================================\n");
	}	 }
	
	

void 	delete_old_records(MonitorSystem *sys, int days){//删除过期记录 
	if (sys == NULL || days <= 0) {
        printf("参数错误！\n");
        return;
    }
    time_t now = time(NULL);
    int delete_count = 0;
    int i;
	for (i=1; i<sys->field_count; i++) {
        Field *field = sys->fields[i];
        SensorRecord *prev = NULL;
        SensorRecord *p = field->head;
        while (p != NULL) {
            struct tm rec_tm = {0};
            scanf(p->timestamp, "%d-%d-%d %d:%d:%d",&rec_tm.tm_year, &rec_tm.tm_mon, &rec_tm.tm_mday,&rec_tm.tm_hour, &rec_tm.tm_min, &rec_tm.tm_sec);
            rec_tm.tm_year -= 1900;
            rec_tm.tm_mon -= 1;
            time_t rec_time = mktime(&rec_tm);
            double diff_days = difftime(now, rec_time) / (24 * 3600);
            if (diff_days > days) {
                if (prev == NULL) {
                    field->head = p->next;
                } else {
                    prev->next = p->next;
                }
                if (p == field->tail) {
                    field->tail = prev;
                }
                SensorRecord *temp = p;
                p = p->next;
                free(temp);
                delete_count++;
                field->sensor_count--;
            } else {
                prev = p;
                p = p->next;
            }
        }
    }
    printf("已删除过期记录共%d条\n", delete_count);
} 

int get_record_count(MonitorSystem *sys, int field_id){//获取记录数量，返回该田块的总记录数
	printf("id为%d的田块有%d条传感器记录\n",field_id,sys->fields[field_id]->sensor_count); 
}
float calculate_avg_temperature(MonitorSystem *sys, int field_id){//计算平均温度
	int i=1;
	float sum=0;
	float avg=0;
	SensorRecord *p=sys->fields[field_id]->head;
	for(i=1;i<=sys->fields[field_id]->sensor_count;i++){
		if(p->field_id==field_id){
			sum+=p->temperature ;
		}
		p=p->next;
	}
	avg=sum/sys->fields[field_id]->sensor_count;
	return avg;
} 

//数据分析函数 
void find_abnormal_records(MonitorSystem *sys, int field_id){//?内存问题 
	if(sys->fields[field_id]==NULL||sys->fields[field_id]->sensor_count==0){
		printf("ID为%d的田块不存在或无传感器数据！\n",field_id);
		return; 
	}
	else{        
		int abnormal_count = 0;
		printf("---id为%d田块的异常记录---\n",field_id);
		printf("原因\t\t温度\t湿度\t记录时间\n") ;
		SensorRecord *p=sys->fields[field_id]->head;
		while(p!=NULL){
			if(p->is_abnormal==1){
				abnormal_count++; 
			if(p->temperature <10){
				printf("温度过低\t");
			}
			if(p->temperature >35){
				printf("温度过高\t"); 
			}
			if(p->humidity >85){
				printf("湿度过高\t");
			}
			if(p->humidity <40){
				printf("湿度过低\t");
			}
			printf("%.2f℃\t%.2f%%\t%s\n",p->temperature,p->humidity,p->timestamp); 
		}
			p=p->next;
		}
		if (abnormal_count==0) {
			printf("该田块无异常记录！\n"); }
		printf("-----------------------\n");
		
	}}
    
void find_extreme_values(MonitorSystem *sys, int field_id)//查找极值：输出最高/最低温度和湿度 
{ 
	SensorRecord *pq=sys->fields[field_id]->head ;
	float max_tem=sys->fields[field_id]->head->temperature ;
	float min_tem=sys->fields[field_id]->head->temperature ;
	float max_humidity=sys->fields[field_id]->head->humidity;
	float min_humidity=sys->fields[field_id]->head->humidity;
	int i=1;
	if(sys->fields[field_id]->head==NULL){
		printf("该区块无信息！");
	}
	else{
		for(i=1;i<sys->fields[field_id]->sensor_count;i++){
		if(pq->temperature>max_tem ){
			max_tem=pq->temperature ;
		}
		if(pq->temperature<min_tem ){
			min_tem=pq->temperature ;
		}
		if(pq->humidity >max_humidity ){
			max_humidity=pq->humidity  ;
		}
		if(pq->humidity <min_humidity ){
			min_humidity=pq->humidity  ;
		}
		pq=pq->next ;
	}
		printf("【最高温度】%.2f℃\n",max_tem);
		printf("【最低温度】%.2f℃\n",min_tem);
		printf("【最高湿度】%.2f%%\n",max_humidity);
		printf("【最低湿度】%.2f%%\n",min_humidity);
	}
	printf("\n");
	}
	
void analyze_trend(MonitorSystem *sys, int field_id){//判断趋势：判断上升/下降/稳定趋势
//分析第一条和最后一条温度，是上升还是下降，湿度上升还是下降. 
if(sys->fields[field_id]->sensor_count<2){
	printf("数据小于两条！无法分析！");
}
else{
	printf("田块：%s(ID:%d)的趋势分析\n",sys->fields[field_id]->name ,field_id);
	printf("==========================\n");
	printf("基于最近%d条记录分析：\n",sys->fields[field_id]->sensor_count);
	//温度 
	if(sys->fields[field_id]->head->temperature < sys->fields[field_id]->tail ->temperature){
		printf("【温度趋势】温度呈上升趋势！\n");
	}
	
	else if(sys->fields[field_id]->head->temperature>sys->fields[field_id]->tail ->temperature) {
		printf("【温度趋势】温度呈下降趋势！\n");
	}
	else{
		printf("【温度趋势】基本稳定（波动%.1f℃）\n",fabs(sys->fields[field_id]->head->temperature-sys->fields[field_id]->tail ->temperature));
	} 
	
	if(sys->fields[field_id]->head->humidity  < sys->fields[field_id]->tail->humidity ){
	printf("【湿度趋势】湿度呈上升趋势!\n"); 
	}
	else if(sys->fields[field_id]->head->humidity  > sys->fields[field_id]->tail->humidity ){
	printf("【湿度趋势】湿度呈下降趋势!\n"); 
	}
	else{
		printf("【湿度趋势】基本稳定（波动%.1f%%）\n",fabs(sys->fields[field_id]->head->humidity-sys->fields[field_id]->tail->humidity)); 
	}
}
	printf("\n");	
} 
//告警管理系统
void check_and_generate_alerts(MonitorSystem *sys, int field_id){//检查并生成告警 
	if(field_id>sys->field_count){
		printf("田块id超出范围");
	} 
	if(sys==NULL||field_id<1||sys->fields[field_id]==NULL){
		printf("田块ID不合法！"); 
		return;
	}
	SensorRecord *b=sys->fields[field_id]->head;//找传感器
	if(b==NULL){
		printf("未查询到相应信息！");
		return;
	} 
	int n=0;//统计告警次数
	while(b!=NULL){
		if(b->is_abnormal==1){
		n++;
		Alert *q=(Alert*)malloc(sizeof(Alert));//传告警
		q->field_id=field_id;
		q->next=NULL;
		getcurrenttime(q->alert_time);
		char type[50]={0};
		 
		if(b->temperature<10){
			strcat(type,"低温告警（2级）、");
			q->severity=2;
		}
		if(b->temperature >35){
			strcat(type,"高温告警（2级）、");
			q->severity=2;
		}
		if (b->humidity >85){
			strcat(type,"高湿告警（1级）、");
			q->severity=1; 
		}
		if(b->humidity <40){
			strcat(type,"低湿告警（1级）、");
			q->severity=1;
		}
		if(strlen(type)>0){
			if(type[strlen(type)-1]=='、'){
				type[strlen(type)-1]='\0';
			} 
		}
		strcpy(q->alert_type,type); 
		if(sys->fields[field_id]->head1==NULL){
			sys->fields[field_id]->head1=q;
			sys->fields[field_id]->tail1=q;
			printf("当前田块id%d\n",sys->fields[field_id]->id);
		}
		else{
			sys->fields[field_id]->tail1->next=q;
			sys->fields[field_id]->tail1=q;  
		}
		printf("田块%d的【%s】告警已生成!\n",field_id,q->alert_type );
		}
		b=b->next;
	} 
		if(n>=3){
		Alert *news=(Alert*)malloc(sizeof(Alert));
		if(news==NULL){
			printf("三级告警内存分配失败！");
			return;
		} 
		news->field_id=field_id;
		getcurrenttime(news->alert_time );
		strcpy(news->alert_type,"持续异常告警（3级）");
		news->severity=3;
		news->next=NULL;
		sys->fields[field_id]->tail1->next=news;	
		sys->fields[field_id]->tail1=news;
		printf("ID为%d的田块3级告警已生成！\n",field_id);
		} 
	}
	
	
 
void display_all_alerts(MonitorSystem *sys) {//显示所有告警 
	int i=1;
	int n=0;
	if(sys==NULL){
		printf("Not found!\n");
	}
	else{
	Alert *list[1000];
	int alert_count=0;
	for(i=1;i<sys->field_count;i++){
		Alert *pq=sys->fields[i]->head1;
		while(pq!=NULL){
			list[alert_count++]=pq;
			pq=pq->next;
		}
	} 
	int m=0;
	for(m=0;m<alert_count-1;m++){
		for(n=0;n<alert_count-m-1;n++){
			if(list[n]->severity>list[n+1]->severity){
				Alert *temp=list[n];
				list[n]=list[n+1];
				list[n+1]=temp;
			}
		}
	}
	printf("========================================\n");
	printf("               所有系统告警\n");
	printf("========================================\n");
	printf("田块ID\t时间            \t类型\t严重程度\t\t描述\n");
	printf("---------------------------------------\n");
	if(sys->record_head!=NULL){		
		int k=0;
		for(k=0;k<alert_count;k++){
			Alert *pq=list[k];
			char serverity_s[20];
			switch(pq->severity){
		case 1:strcpy(serverity_s,"轻微");break;
		case 2:strcpy(serverity_s,"中等");break;
		case 3:strcpy(serverity_s,"严重");break;
		default:strcpy(serverity_s,"未知");
		}
		printf("%d\t",pq->field_id );
		printf("%s\t",pq->alert_time );
		printf("%s\t",pq->alert_type);
		printf("%s \t",serverity_s);
		printf("%s\n",pq->description );
		pq=pq->next ;
		}
		printf("----------------------------------------\n");} 	
	else{
		printf("暂无告警信息\n");
	}
	printf("\n");
	}}
	

int resolve_alert(MonitorSystem *sys, int field_id, const char *alert_type) {//处理告警 删除告警 //如何模糊搜索 
	Alert *q=sys->fields[field_id]->head1;
	Alert *prev=NULL;
	int  count=0;
	while(q!=NULL){
		if(strcmp(q->alert_type ,alert_type)==0){//比较必须使用strcmp,不使用==！第二次错误 
			count++;
			if(prev==NULL){
				sys->fields[field_id]->head1=q->next;		
			}
			else{
				prev->next=q->next;
				if(q==sys->fields[field_id]->tail1){
					sys->fields[field_id]->tail1=prev;
				} 
			}
			Alert *temp=q;
			q=q->next ;
			free(temp);}
			
		else{
			prev=q;
			q=q->next ;
		}
	}
	printf("已删除ID为%d的田块的%d条告警信息！",field_id,count);
	return count;
}
int get_alert_count(MonitorSystem *sys, int field_id) {//获取告警数量 
	int count=0;
	struct Alert *q=(Alert*)malloc(sizeof(Alert));
	q=sys->fields[field_id]->head1;
	while(q!=sys->fields[field_id]->tail1->next)
	{
	count++;
	q=q->next;
	}
	return count;
}
int main(){	
	//*manager=MonitorSystem->Manager;
	MonitorSystem *sys=create_monitor_system();
	int select0=0;//用户输入选择0
	int select1=0;//用户输入选择1 
	while(1){
		display_main_menu ();
		scanf("%d",&select0);
		switch(select0){
			case 1://实验田管理 
			while(1){
			printf("---实验田管理---\n"); 
			printf("1.添加新田块\n"); 
			printf("2.显示所有田块\n");
			printf("3.查找田块（按ID）\n");
			printf("4.搜索田块（按名称）\n");
			printf("5.搜索田块（按负责人）\n");	
			printf("6.删除田块\n");
			printf("0.返回主菜单\n");
			printf("请选择：");
			scanf("%d",&select1);
			if(select1==1){//添加新田块		
				printf("请输入田块负责人");
				const char manager[20];
				scanf("%s",manager);
				add_field(sys,manager);
				}
			else if(select1==2){ //显示所有田块
				display_all_fields(sys);
				}
		 	else if(select1==3){ //查找田块（按ID）
		 		int field_id;
				printf("请输入查找的id:");
				scanf("%d",&field_id); 
		 		find_field_by_id(sys,field_id);
				}
			else if(select1==4){ //搜索田块（按名称）
				const char keyword[20];
				printf("请输入查找的名称：");
				scanf("%s",&keyword);
				search_fields_by_name(sys,keyword);
			}
			else if(select1==5){// 搜索田块（按负责人）
			const char manager_name[20];
			printf("请输入查找的负责人：");
			scanf("%s",&manager_name);
			search_fields_by_manager(sys, manager_name);	}
			else if(select1==6){ // 删除田块
			int field_id=0;
			printf("请输入要删除的田块ID:"); 
			scanf("%d",&field_id);
			delete_field(sys,field_id);
				}
			else if(select1==0){ //返回主菜单
			printf("返回主菜单...\n");
				break;}	
			else{
				printf("输入错误，请重新输入。\n");
			}}
			break;
			case 2://传感器数据管理 
				
				while(1){
				printf("---传感器数据管理---\n");
				printf("1.添加传感器记录\n");
				printf("2.显示最新记录\n");
				printf("3.显示所有记录\n");
				printf("4.删除过期记录\n");
				printf("5.获取记录数量\n");
				printf("6.计算平均温度\n"); 
				printf("0.返回主菜单\n");
			printf("请选择：");
			scanf("%d",&select1);
			if(select1==1){//1.添加传感器记录 
				printf("请输入对应田块ID:");
				int field_id=0;
				scanf("%d",&field_id);
				if(field_id<=sys->field_count&&field_id>=1){
				printf("请输入对应温度:");
				float temperature;
				scanf("%f",&temperature);
				printf("请输入对应湿度:");
				float humidity;
				scanf("%f",&humidity);
				add_sensor_record(sys, field_id, temperature,humidity);
				}
				else{
						printf("无此id田块！\n");
					}	 
				}
			else if(select1==2){ //2.显示最新记录
				int field_id;
				int count;
				printf("请输入要查询的田块id:");
				scanf("%d",&field_id);
				printf("请输入要查询最新几条记录");
				scanf("%d",&count);
				display_latest_records(sys, field_id, count);
				}
		 	else if(select1==3){ //3.显示所有记录
		 		printf("请输入查询id:");
		 		int field_id;
		 		scanf("%d",&field_id); 
		 		display_field_all_records(sys, field_id) ; 
				}
			else if(select1==4){//4.删除过期记录
			int days;
			printf("请输入要保留最近几天的数据\n");
			scanf("%d",&days);
			delete_old_records(sys, days); 
			}
			else if(select1==5){//5.获取记录数量
				printf("请输入查询id:");
		 		int field_id;
		 		scanf("%d",&field_id); 
				get_record_count(sys, field_id); 
			}
			else if(select1==6){//6.计算平均温度
				float avg_temperature=0;
				printf("请输入查询id:");
		 		int field_id;
		 		scanf("%d",&field_id);
				avg_temperature=calculate_avg_temperature(sys, field_id);
				printf("平均温度为%.2f\n",avg_temperature); 
			} 
			else if(select1==0){ //返回主菜单
				printf("返回主菜单...\n");
				break; }	
			else{
				printf("输入错误，请重新输入。\n");
			}}
				break;
		 	case 3://数据查询与统计 
		 		
				while(1){
				printf("---数据查询与统计---\n");
				printf("1.查找异常记录\n");
				printf("2.查找极值\n");
				printf("3.分析趋势\n");
				printf("0.返回主菜单\n");
			printf("请选择：");
			scanf("%d",&select1);
			if(select1==1){//1.查找异常记录
				printf("请输入田块ID");
				int field_id=1;
				scanf("%d",&field_id);
				find_abnormal_records(sys, field_id); 
				} 
			else if(select1==2){ //2.查找极值
			int field_id=1;
			printf("请输入要查找极值的田块id");
			scanf("%d",&field_id);
			find_extreme_values(sys, field_id); 
				}
		 	else if(select1==3){ //3.分析趋势
		 		int field_id;
				printf("请输入田块id分析趋势");
				scanf("%d",&field_id); 
		 		analyze_trend(sys, field_id);
				}
			else if(select1==0){ //返回主菜单
				printf("返回主菜单...\n");
				break; }	
			else{
				printf("输入错误，请重新输入。\n");
			}}
				break;
			case 4://告警系统 
				
				while(1){
					printf("---告警管理---\n");
					printf("1.检查并生成告警\n");
					printf("2.显示所有告警\n");
					printf("3.处理告警\n");
					printf("4.统计告警数量\n");
					printf("0.返回主菜单\n");
					printf("请输入选择：");
					scanf("%d",&select1);
					if(select1==1){
							int field_id;
							printf("请输入搜索id");
							scanf("%d",&field_id);
							check_and_generate_alerts(sys, field_id); 
							continue;}
					else if(select1==2){
						display_all_alerts(sys); 
						continue;
					}
					else if(select1==3){
						int field_id;
						printf("请输入搜索id");
						scanf("%d",&field_id);
						const char*alert_type;
						printf("请输入要删除的告警信息类型：");
						scanf("%s",alert_type); 
						resolve_alert(sys, field_id, alert_type);
						continue;
					}	
							
					else if(select1==4){
						int field_id;
						int count=0; 
						printf("请输入搜索id");
						scanf("%d",&field_id);
						count=get_alert_count(sys, field_id);
						printf("该id所对应田块共有%d条告警信息\n",count);
					}		
					else if(select1==0){
						printf("返回主菜单...\n");
						break; 
					}	
						}
					break;				 
		 	case 5://系统信息 
		 		printf("==============================================\n"); 
		 		printf("本1.0系统由计科251班高晓雪(学号：19225126)制作\n");
		 		printf("qq:2338177148\n");
				printf("==============================================\n"); 
				break;
			case 0://退出系统
				destroy_monitor_system(sys);
				return 0;
				break; 
			default:
				printf("输入错误，请重新输入。\n"); 
				break;
		 }}}
