出题方代码：
/*!
 * MindPlus
 * mpython
 *
 */
#include <MPython.h>
#include <DFRobot_Iot.h>
#include <mPython_tinywebdb.h>

// 动态变量
String mind_s_TiMu;
// 函数声明
void onButtonAPressed();
void obloqMqttEventT1(String& message);
void obloqMqttEventT2(String& message);
// 静态常量
const String topics[5] = {"D46IXy_Gg","qIbXXs_Gg","6XvyXWqGg","YJkimG3Mg",""};
const MsgHandleCb msgHandles[5] = {NULL,obloqMqttEventT1,obloqMqttEventT2,NULL,NULL};
// 创建对象
DFRobot_Iot       myIot;
mPython_TinyWebDB mydb;


// 主程序开始
void setup() {
	mPython.begin();
	dfrobotRandomSeed();
	myIot.setMqttCallback(msgHandles);
	buttonA.setPressedCallback(onButtonAPressed);
	myIot.wifiConnect("602iot", "18wulian");
	while (!myIot.wifiStatus()) {yield();}
	display.setCursorLine(1);
	display.printLine("WiFi连接成功");
	myIot.init("iot.dfrobot.com.cn","wIG1ySCMR","","QSMJsSCGRz",topics,1883);
	myIot.connect();
	while (!myIot.connected()) {yield();}
	display.setCursorLine(2);
	display.printLine("MQTT连接成功");
	mydb.setServerParameter("http://tinywebdb.appinventor.space/api", "testdb","e5332d47");
	display.setCursorLine(3);
	display.printLine("按下A键出题");
}
void loop() {

}

// 事件回调函数
void onButtonAPressed() {
	mind_s_TiMu = mydb.getTag((String((random(1, 3+1)))));
	myIot.publish(topic_0, mind_s_TiMu);
	display.fillScreen(0);
	display.setCursorLine(1);
	display.printLine("题目：");
	display.setCursorLine(2);
	display.printLine(mind_s_TiMu);
	display.setCursorLine(3);
	display.printLine("等待抢答……");
}
void obloqMqttEventT1(String& message) {
	display.fillScreen(0);
	display.setCursorLine(1);
	display.printLine((String(message) + String("抢答成功")));
	display.setCursorLine(2);
	display.printLine((String((String("等待") + String(message))) + String("回答……")));
}
void obloqMqttEventT2(String& message) {
	display.setCursorLine(2);
	display.printLine((String("回答的答案为") + String(message)));
	display.setCursorLine(3);
	display.printLine("按A键重新出题");
}

抢答方代码：
/*!
 * MindPlus
 * mpython
 *
 */
#include <MPython.h>
#include <DFRobot_Iot.h>

// 动态变量
volatile float mind_n_YiBeiQiangDa, mind_n_QiangDaZhuangTai;
// 函数声明
void obloqMqttEventT0(String& message);
void onButtonAPressed();
void onButtonBPressed();
// 静态常量
const String topics[5] = {"D46IXy_Gg","qIbXXs_Gg","6XvyXWqGg","",""};
const MsgHandleCb msgHandles[5] = {obloqMqttEventT0,NULL,NULL,NULL,NULL};
// 创建对象
DFRobot_Iot myIot;


// 主程序开始
void setup() {
	mPython.begin();
	myIot.setMqttCallback(msgHandles);
	buttonA.setPressedCallback(onButtonAPressed);
	buttonB.setPressedCallback(onButtonBPressed);
	myIot.wifiConnect("602iot", "18wulian");
	while (!myIot.wifiStatus()) {yield();}
	display.setCursorLine(1);
	display.printLine("WiFi连接成功");
	myIot.init("iot.dfrobot.com.cn","wIG1ySCMR","","QSMJsSCGRz",topics,1883);
	myIot.connect();
	while (!myIot.connected()) {yield();}
	display.setCursorLine(2);
	display.printLine("MQTTl连接成功");
	display.setCursorLine(3);
	display.printLine("等待出题……");
	mind_n_YiBeiQiangDa = 0;
}
void loop() {

}

// 事件回调函数
void obloqMqttEventT0(String& message) {
	mind_n_YiBeiQiangDa = 0;
	display.fillScreen(0);
	display.setCursorLine(1);
	display.printLine((String("题目：") + String(message)));
	display.setCursorLine(2);
	display.printLine("按下B键抢答");
}
void onButtonAPressed() {
	if (((String(mind_n_QiangDaZhuangTai).toInt())==1)) {
		myIot.publish(topic_2, "YES");
		display.setCursorLine(3);
		display.printLine("您的答案为YES");
		display.fillInLine(4, 0);
		mind_n_QiangDaZhuangTai = 0;
	}
}
void onButtonBPressed() {
	if (((String(mind_n_YiBeiQiangDa).toInt())==0)) {
		display.setCursorLine(2);
		display.printLine("抢答成功！");
		mind_n_QiangDaZhuangTai = 1;
		mind_n_YiBeiQiangDa = 1;
		myIot.publish(topic_1, "梁庭锋");
		display.setCursorLine(3);
		display.printLine("按A键回答YES");
		display.setCursorLine(4);
		display.printLine("B键回答NO");
	}
	else {
		if (((String(mind_n_QiangDaZhuangTai).toInt())==0)) {
			display.setCursorLine(3);
			display.printLine("抢答失败！题目已被抢答");
		}
		else {
			myIot.publish(topic_2, "NO");
			mind_n_QiangDaZhuangTai = 0;
			display.setCursorLine(3);
			display.printLine("您的答案为NO");
			display.fillInLine(4, 0);
		}
	}
}
