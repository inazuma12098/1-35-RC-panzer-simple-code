# 1-35-RC-panzer-simple-code

//注意事項
//這邊只有行走部分程式碼
//必須在A0,A1都街上一個接收器的訊號，否則會有非常大的延遲
//無調整剛對機，搖控器搖桿往上那個通道的數值會變大
//往左也是變大
//通常在900~2100的範圍內，中立點為1500左右
//這邊的程式碼上傳後可以在監控視窗看到數值
//每個通道數值會跳動是正常的

int receiver1=A0;//宣告接收器訊號位置，通常我使用A0~A5共六個通道，行走只需兩個
int receiver2=A1;//宣告接收器訊號位置

//馬達腳位，由於可以變速度的腳位只有D3\D5\D6\D9\D10\D11
//所以需要可以變速的話必須在這幾隻腳位上
int motor1a=11;//宣告馬達訊號位置(這邊只宣告腳位，正式開啟輸出在下面的setup裡面，若下面沒有寫成OUTPUT，會視為有個變數motor1a他的值為11這樣)
int motor1b=10;//宣告馬達訊號位置
int motor2a=9;//宣告馬達訊號位置
int motor2b=5;//宣告馬達訊號位置

int walkflag;//宣告是否為行走flag，之後可以設條件給開砲的時候車身震動用，這邊是不會用到，但已經寫好行走的時候這個變數就為1了

void setup() {//上電時執行的設定部分
  Serial.begin(9600); 
  pinMode(receiver1,INPUT);//設定為輸入，預設就是輸入，可以省略
  pinMode(receiver2,INPUT);//設定為輸入，預設就是輸入，可以省略
  pinMode(motor1a,OUTPUT);//設定為輸出，馬達、伺服馬達訊號、LED等的腳位需設定為輸出
  pinMode(motor1b,OUTPUT);//設定為輸出
  pinMode(motor2a,OUTPUT);//設定為輸出
  pinMode(motor2b,OUTPUT);//設定為輸出
}

void loop() {//正式執行的部分
  
  //宣告“訊號”變數，名字可以自訂(但要以unsigned long宣告)
  unsigned long signal1,signal2;
  
  //宣告調整轉速用的變數，名字也可以自取
  int r1,r2;
  
  signal1 = pulseIn(receiver1, HIGH);//讀取前面設定的A0腳位的脈衝訊號，訊號的高電位時間轉換成數值(航模遙控器的值通常在900~2100，若無調整1500為中立值)
  signal2 = pulseIn(receiver2, HIGH);//讀取前面設定的A1腳位的脈衝訊號
  
  Serial.println(String("")+"訊號1 = "+ signal1+" 訊號2 = "+ signal2);//這邊使我們可以在監視視窗看到值的變化，可以在確認之後刪除，若要觀看其他值，可以把()裡的signal1或signal2換掉


//下方以if...else if...else的方式寫成條件
//只要滿足其中一項()中的條件，即會執行該{}中的動作而不做其他動作
//在判斷式中 &&代表而且 ||代表或 ==代表等於

//在C語言中，若只打=是代表指定為 如：Ａ＝1; 是讓變數Ａ變成1
//要判斷是否等於則是 if(A==1){} 此時若A等於1，執行{}裡的所有動作

      
if(signal1<900){//條件一：防止接收器開了但遙控器沒開時暴衝，在這個狀況讀取都為0在一般使用時的訊號範圍外，馬達強制設定低電位
      digitalWrite(motor1a,LOW);//馬達訊號輸出低電位，若要高電位將LOW改成HIGH即可
      digitalWrite(motor1b,LOW);//馬達訊號輸出低電位
      digitalWrite(motor2a,LOW);//馬達訊號輸出低電位
      digitalWrite(motor2b,LOW);//馬達訊號輸出低電位
        
    }
    
else if(signal1<1550&&signal1>1450&&signal2>1450&&signal2<1550){//條件二：在搖桿位置在這個範圍時做以下動作：停止
            
       walkflag=0;//未行走，此變數設為0;
       digitalWrite(motor1a,LOW);//馬達訊號輸出低電位
       digitalWrite(motor1b,LOW);//馬達訊號輸出低電位
       digitalWrite(motor2a,LOW);//馬達訊號輸出低電位
       digitalWrite(motor2b,LOW);//馬達訊號輸出低電位
     }
                
else if(signal1>1550&&signal2>1450&&signal2<1550){//條件三：前進
      
        walkflag=1; //行走，設為1
        
        r1=(signal1-1550)*255/350;  //以讀取的值轉換成馬達用訊號0~255，255為最高速
        
        if(r1>255){r1=255;} //為避免有調整過超出範圍導致錯誤，以條件限制在0~255
        if(r1<0){r1=0;}     //同上
        
        analogWrite(motor1a,r1);  //輸出上方數值給馬達
        analogWrite(motor1b,0);   //上一行馬達的另一隻接腳的訊號
        analogWrite(motor2a,r1);   //輸出上方數值給馬達
        analogWrite(motor2b,0);   //上一行馬達的另一隻接腳的訊號
        } 

else if(signal1>1550&&signal2>1550){//條件四：前左
      
        walkflag=1;
        
        r1=(signal1-1550)*255/350;
        if(r1>255){r1=255;}
        if(r1<0){r1=0;}
        
        r2=150-(signal2-1550)*150/350;//慢速履帶的轉速調整
        if(r2>150){r2=150;}           //搖桿越往左側，此馬達越慢
        if(r2<0){r2=0;}               
        analogWrite(motor1a,r2);
        analogWrite(motor1b,0);
        analogWrite(motor2a,r1);
        analogWrite(motor2b,0);
        } 


else if(signal1>1550&&signal2<1450){//條件五：前右
        walkflag=1;
        r1=(signal1-1550)*255/350;
        if(r1>255){r1=255;}
        if(r1<0){r1=0;}
        r2=150-(1450-signal2)*150/350;//慢速履帶的轉速調整
        if(r2>150){r2=150;}           //搖桿越往右側，此馬達越慢
        if(r2<0){r2=0;}
        analogWrite(motor1a,r1);
        analogWrite(motor1b,0);
        analogWrite(motor2a,r2);
        analogWrite(motor2b,0);
        } 


else if(signal1>1450&&signal1<1550&&signal2>1550){//條件六：左原地迴轉
        walkflag=1;
        r1=(signal2-1550)*127/350;
        if(r1>127){r1=127;}
        if(r1<0){r1=0;}
        analogWrite(motor1a,0);
        analogWrite(motor1b,r1);      //將一側馬達訊號互換，變為一顆正轉一顆反轉
        analogWrite(motor2a,r1);
        analogWrite(motor2b,0);
        } 

      
else if(signal1>1450&&signal1<1550&&signal2<1450){//條件七：右原地迴轉
        walkflag=1;
        r1=(1450-signal2)*127/350;
        if(r1>127){r1=127;}
        if(r1<0){r1=0;}
        analogWrite(motor1a,r1);
        analogWrite(motor1b,0);     //將上個條件馬達訊號互換，變為一顆正轉一顆反轉
        analogWrite(motor2a,0);
        analogWrite(motor2b,r1);
        } 
      
else if(signal1<1450&&signal2>1450&&signal2<1550){//條件八：後退
        walkflag=1;
        r1=(1450-signal1)*255/350;
        if(r1>255){r1=255;}
        if(r1<0){r1=0;}
        analogWrite(motor1a,0);
        analogWrite(motor1b,r1);
        analogWrite(motor2a,0);
        analogWrite(motor2b,r1);
        } 
      
else if(signal1<1450&&signal2>1550){//條件九：後左
        walkflag=1;
        r1=(1450-signal1)*255/350;
        if(r1>255){r1=255;}
        if(r1<0){r1=0;}
        r2=117-(signal2-1550)*117/350;
        if(r2>117){r2=117;}
        if(r2<0){r2=0;}
        analogWrite(motor1a,0);
        analogWrite(motor1b,r2);
        analogWrite(motor2a,0);
        analogWrite(motor2b,r1);
        } 
      
else if(signal1<1450&&signal2<1450){//條件十：後右
        walkflag=1;
        r1=(1450-signal1)*255/350;
        if(r1>255){r1=255;}
        if(r1<0){r1=0;}
        r2=117-(1450-signal2)*117/350;
        if(r2>117){r2=117;}
        if(r2<0){r2=0;}
        analogWrite(motor1a,0);
        analogWrite(motor1b,r1);
        analogWrite(motor2a,0);
        analogWrite(motor2b,r2);
        }             
            
}
