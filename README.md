# EmbeddedSystemsMazeSolver
# A sample of code for my 3rd year Embedded Systems Module, Code runs on a lego mindstorm allowing it to solve a maze and remove dead ends
# fixed formatting

sensor configuration
```
#pragma config(Sensor, S1, Gyro, sensorEV3_Gyro, modeEV3Gyro_RateAndAngle)
#pragma config(Sensor, S2,     Colour,         sensorEV3_Color, modeEV3Color_Reflected)
#pragma config(Sensor, S4,     Touch,          sensorEV3_Touch)
#pragma config(Sensor, S3,     IRDistance1,    sensorEV3_IRSensor, modeEV3IR_Proximity)
#pragma config(Motor,  motorA,          motorLeft,     tmotorEV3_Large, PIDControl, encoder)
#pragma config(Motor,  motorD,          motorRight,    tmotorEV3_Large, PIDControl, encoder)
```
initalising variables with inital values
```
int currentDistance = 0;
int currentLight = 0;
int turn = 80;
int threshold = 300;
int floorColour;
bool inMaze = true;
string route[100];
int pos = 0;
bool deadEnd = false;
int depth = 3;
string currentPos;
```
task 1 distance reading (IR camera to wall)
```
task distanceReading() {
	while (true) {
		currentDistance = SensorValue[IRDistance1];
		//displayCenteredBigTextLine(4, "Dist: %d", currentDistance);
	}
}
```
task 2 light reading (looking at floor colour to find maze end)
```
task lightReading() {
	while (true) {
		currentLight = SensorValue[Colour];
		//displayCenteredBigTextLine(8, "Light: %d", currentLight);
	}
}
```
task 3 list pos (check the list of turns)
```
task listPos() {
	while (true) {
		currentPos = route[pos];
		displayCenteredBigTextLine(4,"List: %s", currentPos);
	}
}
```
function 1 deadend check (check for deadends)
```
void deadEndCheck (){
	if((route[pos]=="R")&&(route[pos-1]=="R")){
		deadEnd = true;
	}
}
```
function 2 eliminate route (mark a path as being a dead end)
```
void eliminateRoute (){
	if(route[pos]!=route[pos-depth]){
		depth += 2;
	}else{
		for(int i=0;i<depth;i++){
				route[pos-i]="";
				displayCenteredBigTextLine(2, "eliminating end");
		}
		route[pos-depth]="S";
		pos = pos-depth;
		pos ++;
		depth = 2;
		deadEnd = false;
	}
}
```
function 3 turn right (turn 90' clockwise)
```
void turnRight(){
		resetGyro(Gyro);
		while(SensorValue[Gyro] < turn){
			motor[motorLeft] = 20;
			motor[motorRight] = -20;
			displayCenteredBigTextLine(2, "Turning Right");
		}
		resetGyro(Gyro);
		route[pos] = "R";
		if(deadEnd == false ){
			deadEndCheck();
		}else{
			eliminateRoute();
		}
		pos++;
}
```
function 4 forward (move forward)
```
void forward(){
		motor[motorLeft] = 20;
		motor[motorRight] = 20;
		displayCenteredBigTextLine(2, "Moving foward");
}
```
function 5 halt (stops the robot)
```
void halt(){
		motor[motorLeft] = 0;
		motor[motorRight] = 0;
		displayCenteredBigTextLine(2, "halted");
}
```
function 6 turn left (turn 90' counter clockwise)
```
void turnLeft(){
		wait1Msec(100);
		resetGyro(Gyro);
		while(SensorValue[Gyro] > -turn){
				//displayCenteredBigTextLine(2, "Turning Left");
				motor[motorLeft] = 0;
				motor[motorRight] = 20;

			}
		displayCenteredBigTextLine(2, "Finished turning");
		resetGyro(Gyro);
		route[pos] = "L";
		if(deadEnd == true){
			eliminateRoute();
		}
		pos++;
}
```
main branch
```
task main()
{
	startTask(distanceReading);
	startTask(lightReading);
	startTask(listPos);
	while(true){
		while(inMaze){

			// Read the sensor

			//displayCenteredBigTextLine(4, "Dist: %d", currentDistance);
			while((!SensorValue[Touch])&&(inMaze == true)){
				forward();
				floorColour = SensorValue[Colour];
				if (floorColour > 10){
					inMaze = false;
					//displayCenteredBigTextLine(6, "BREAK");
				} else {
					if (currentDistance > threshold) {
						wait1Msec(150);
						if(route[pos]=="S"){
							pos ++;
							displayCenteredBigTextLine(2, "Pos is S");
							while(currentDistance > threshold){
								forward();
							}
						}else{
							displayCenteredBigTextLine(2, "Pos is NOT S");
							turnLeft();
						}
						while ( currentDistance > threshold ) {
							displayCenteredBigTextLine(2, "Looping");
							resetGyro(Gyro);
							forward();
						}
					} else if (currentDistance > 70 ){
						resetGyro(Gyro);
						motor[motorLeft] = 10;
						motor[motorRight] = 25;
					} else {
						resetGyro(Gyro);
						motor[motorLeft] = 25;
						motor[motorRight] = 10;
					}
				}
			}
			if (inMaze == true){
				motor[motorLeft] = -15;
				motor[motorRight] = -15;
				wait1Msec(800);
				turnRight();
			}else{
				halt();
				for(int i=0;i<=pos;i++){
					displayCenteredBigTextLine(i+1, route[i]);
				}
				inMaze = true;
				pos = 0;
				wait1Msec(5000);
			}
		}
	}
}

```
