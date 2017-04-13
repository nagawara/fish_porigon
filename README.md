//fish_porigon
//魚の群体制御を行うCGでございます

//ilink32 itec.othello02.config.obj c0x32.obj itec.othello02.playAgent01.obj itec.othello02.playAgent01MinimaxDiffStdio01.obj itec.othello02.minimaxseacher01.obj itec.othello02.util01.obj, playAgent01MinimaxDiffStdio01.exe, , cw32.lib import32.lib
#define __CBOID_H

#define _USE_MATH_DEFINES


#include <iostream>
#include <stdlib.h>
#include <GL/glut.h>
#include <time.h>


int colora = 0, colorb = 0, colorc = 0, colord = 0;
int switchq = 0;
int switchs = 0;

int		gMouseX, gMouseY;
int		gPitch = 30;
int		gYaw;
double	gTranslate = -3.5;
double	dist = -10.0;
double	theta = 0.0;
double sizek = 1;
using namespace std;
void menu(int val);

int kNumBoids = 100;			// The number of fish
double kRoomW = 6;		// width of 'the room'
double kRoomH = 6;		// height of 'the room'
double kRoomD = 6;		// depth of 'the room'

struct CBoid
{
	GLfloat sMaterialBody[4];
	GLfloat sMaterialFin[4];

	double kSight = 0.9;     // radius of sight
	double kFrontCos = -0.9; // lower bound of cosine of the 'front'

	double kDt = 1.0 / 100;    // step time
	double kGain1 = 1;      // separation gain
	double kGain2 = 4;      // alignment gain
	double kGain3 = 5;      // cohesion gain
	double kGainFv = .5;    // viscous friction coefficient
	double kM = 1;          // mass
	double  x, y, z;     // position
	double  vx, vy, vz;  // velocity

};
float eyes_b[] = { 0.0, 0.0, 0.0, 1.0 };
float eyes_w[] = { 1.0, 1.0, 1.0, 1.0 };




void GLUT_SET_MENU()
{
	glutCreateMenu(menu);
	glutAddMenuEntry("SMOOTH", 1);
	glutAddMenuEntry("FLAT", 2);
	glutAddMenuEntry("サイコモード", 3);
	glutAddMenuEntry("普通モード", 4);
	glutAddMenuEntry("魚の大きさを2倍に", 5);
	glutAddMenuEntry("魚の大きさを1/2倍に", 6);
	glutAttachMenu(GLUT_RIGHT_BUTTON);
}





CBoid boid[1000 + 1];
GLfloat sMaterialBody[] = { 0, 1, .3, 1 };
GLfloat sMaterialFin[] = { .5, 0, .5, 1 };

int     counter;     // general counter for miscellaneous use
double  finAngle;    // angle of tail fin
double  hire;
void setPosition(int i, double aX, double aY, double aZ);
void setVelocity(int i, double aVx, double aVy, double aVz);
double getX(double x) { return x; };
double getY(double y) { return y; };
double getZ(double z) { return z; };
double getVx(double vx) { return vx; };
double getVy(double vy) { return vy; };
double getVz(double vz) { return vz; };
void draw(int i, double size);
void rotate(int i);
void translate(int i);


void menu(int val)
{
	switch (val)
	{
	case 1:
		glShadeModel(GL_SMOOTH);
		break;

	case 2:
		glShadeModel(GL_FLAT);
		break;

	case 3:
		switchs = 1;
		break;
	case 4:
		switchs = 0;
		break;
	case 5:
		sizek = sizek * 2;
		break;
	case 6:
		switchs = sizek / 2;
		break;

	}
}

void myIdle(void)
{
	dist += 0.00001;
	if (dist >= -1.0) glutIdleFunc(NULL);
	theta = fmod(theta + 0.000001, 360.0);
	glutPostRedisplay();
}

void srand48(int seed)
{
	srand(seed);
}


double drand48()
{
	return ((double)(rand()) / RAND_MAX);
}

enum _mousemode {//
	eTranslate, eRotate,
}gMouseMode;

enum _viewpoint {
	eFixed = 0, eStalking0, eNViewpoint
}gViewpoint;

void myMouseFunc(int button, int state, int x, int y)
{
	if (button == GLUT_LEFT_BUTTON) {
		gMouseX = x - gYaw;
		gMouseY = y - gPitch;
		gMouseMode = eRotate;
	}
	if (button == GLUT_RIGHT_BUTTON) {
		gMouseY = -y - (int)(gTranslate * 40);
		gMouseMode = eTranslate;
	}
	if (button == GLUT_MIDDLE_BUTTON && state == GLUT_DOWN) {
		gViewpoint = (_viewpoint)((gViewpoint + 1) % eNViewpoint);
	}
}

void myMotionFunc(int x, int y)
{
	if (gMouseMode == eRotate) {
		gYaw = x - gMouseX;
		gPitch = y - gMouseY;
	}
	else if (gMouseMode == eTranslate) {
		gTranslate = (-y - gMouseY) / 40.0;
	}
}

void mySetLight(void)
{
	int	pos[] = { 0,1,-1,0 };
	glLightiv(GL_LIGHT0, GL_POSITION, pos);//照明設定
	glEnable(GL_LIGHT0);//番号0のライトを有効にします
}


void myDisplay(void)
{
	int i;
	const float materialFloor[][4] = { { 0, 1, .8, 1 },{ 0, .8, 1, 1 }, };

	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	glEnable(GL_DEPTH_TEST);
	glEnable(GL_LIGHTING);

	glClear(GL_COLOR_BUFFER_BIT);

	glPushMatrix();
	gluLookAt(0.0, 1.0, dist, 0.0, 1.0, dist + 1.0, 0.0, 1.0, 0.0);
	//xyzAxes(10.0);
	//glPushMatrix();
	glTranslated(1.0, 2.0, 0.0);
	//glRotated(theta, 1.0, 1.0, 0.0);
	//glColor3d(1.0, 0.0, 0.0);
	//glutWireTeapot(1.0);
	glPopMatrix();

	glColor3d(1.0, 1.0, 1.0);		// draw floor
	glBegin(GL_LINES);
	for (i = -5; i< 5; i += 1) {
		glVertex3i(i, 0, -1);
		glVertex3i(i, 0, 1);
		glVertex3i(-1, 0, i);
		glVertex3i(1, 0, i);
	}
	glEnd();



	glPushMatrix();//行列スタックをコピーする
	glTranslatef(0, 0, gTranslate);//(dx,dy,dz)だけ移動
	glRotatef(gPitch, 1, 0, 0);////glRotatefは描画するものを指定したX、Y、Zベクトル軸を中心に回転させる
	glRotatef(gYaw, 0, 1, 0);
	if (gViewpoint == eStalking0) {     	// stalking mode
		glTranslatef(getX(boid[0].x),	// translate the viewpoint to boid #0
			getY(boid[0].y),
			getZ(boid[0].z)
		);
	}
	mySetLight();
	glNormal3f(0, 0, 1);
	for (int z = -(int)kRoomH * 10 / 2; z < (int)kRoomH * 10 / 2 - 1; z += 5) {
		for (int x = -(int)kRoomW * 10 / 2; x < (int)kRoomW * 10 / 2 - 1; x += 5) {
			glMaterialfv(GL_FRONT_AND_BACK,
				GL_AMBIENT_AND_DIFFUSE,
				materialFloor[(((z / 5) & 1) + ((x / 5) & 1)) & 1]
			);
			glBegin(GL_POLYGON);
			glVertex3f(x * .1, -kRoomD / 2, z * .1);
			glVertex3f(x * .1 + .5, -kRoomD / 2, z * .1);
			glVertex3f(x * .1 + .5, -kRoomD / 2, z * .1 + .5);
			glVertex3f(x * .1, -kRoomD / 2, z * .1 + .5);
			glEnd();
		}
	}

	for (int i = 0; i < kNumBoids; i++) {
		glPushMatrix();
		translate(i);
		rotate(i);
		if (switchs == 1)
		{
			sizek = drand48() * 2;
		}
		draw(i, sizek);
		glPopMatrix();
	}
	glPopMatrix();
	glutSwapBuffers();
}

void myReshape(int width, int height)
{
	glViewport(0, 0, width, height);
	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	gluPerspective(90.0, (double)width / (double)height, 0.1, 20.0);
	glMatrixMode(GL_MODELVIEW);
	glLoadIdentity();
}
void myKeyboard(unsigned char key, int x, int y)
{
	int i,m;
	if (key == 'q')exit(0);
	if (key == 'a')sizek = sizek + 0.01;
	if (key == 'b')sizek = sizek - 0.01;
	if (key == 't')switchq = 1;
	if (key == 'y')switchq = 0;

	if (key == 'd')
	{
		for (i = 0; i < kNumBoids; i++)
		{
			boid[i].x = 0;
			boid[i].y = 0;
			boid[i].z = 0;
		}
	}
	if (key == 's')
	{
		for (i = 0; i < kNumBoids; i++)
		{
			m = rand() % kNumBoids;
			boid[m].kDt = boid[m].kDt / 10;
		}
		if (key == 'z')
		{
			for (i = 0; i < kNumBoids; i++)
			{
				m = rand() % kNumBoids;
				boid[m].kDt = boid[m].kDt * 10;
			}
		}
		if (key == 'o')
		{
			for (i = 0; i < kNumBoids; i++)
			{
				//m = rand() % kNumBoids;
				boid[i].kDt = 0.01;
			}
		}
		if (key == 'r')
		{
			for (i = 0; i < kNumBoids; i++)
			{
				boid[i].x = 0;
				boid[i].y = 0;
				boid[i].z = 0.1;
				boid[i].vx = 0.01*M_PI;
				boid[i].vy = 0;
				boid[i].vz = 0.1;
			}
		}
		if (key == 'g')
		{
			for (i = 0; i < kNumBoids; i++)
			{
				boid[i].vx = 1;
				boid[i].vy = 0;
				boid[i].vz = 0;
			}
		}
		if (key == 'l')
		{
			for (i = 0; i < kNumBoids; i++)
			{
				m = rand() % kNumBoids;
				boid[m].vx = 1;
				boid[m].vy = 0;
				boid[m].vz = 0;
			}
		}
	}
}
	void shadowMatrix(GLfloat *m,GLfloat plane[4],GLfloat light[4])
	{
	GLfloat dot;

	// Find dot product between light position vector and ground plane normal.
	dot = plane[0] * light[0] +
		plane[1] * light[1] +
		plane[2] * light[2] +
		plane[3] * light[3];

	m[0] = dot - light[0] * plane[0];
	m[4] = 0.f - light[0] * plane[1];
	m[8] = 0.f - light[0] * plane[2];
	m[12] = 0.f - light[0] * plane[3];

	m[1] = 0.f - light[1] * plane[0];
	m[5] = dot - light[1] * plane[1];
	m[9] = 0.f - light[1] * plane[2];
	m[13] = 0.f - light[1] * plane[3];

	m[2] = 0.f - light[2] * plane[0];
	m[6] = 0.f - light[2] * plane[1];
	m[10] = dot - light[2] * plane[2];
	m[14] = 0.f - light[2] * plane[3];

	m[3] = 0.f - light[3] * plane[0];
	m[7] = 0.f - light[3] * plane[1];
	m[11] = 0.f - light[3] * plane[2];
	m[15] = dot - light[3] * plane[3];
}

void myTimer(int aArg)
{
	double d, fx = 0, fy = 0, fz = 0;
	double dgx = 0;// gx / numBoids - boid[i].x;
	double dgy = 0;// gy / numBoids - boid[i].y;
	double dgz = 0;// gz / numBoids - boid[i].z;
	double dg;// sqrt(dgx * dgx + dgy * dgy + dgz * dgz);
	double numBoids = 0;
	double gx = 0, gy = 0, gz = 0;


	if (switchq == 0)
	{
		if (colora == 0)
			sMaterialBody[0] = sMaterialBody[0] - 0.01;
		if (colora == 1)
			sMaterialBody[0] = sMaterialBody[0] + 0.01;
		if (colorb == 0)
			sMaterialBody[1] = sMaterialBody[1] - 0.01;
		if (colorb == 1)
			sMaterialBody[1] = sMaterialBody[1] + 0.01;
		if (colorc == 0)
			sMaterialBody[2] = sMaterialBody[2] - 0.01;
		if (colorc == 1)
			sMaterialBody[2] = sMaterialBody[2] + 0.01;
		if (colord == 0)
			sMaterialBody[3] = sMaterialBody[3] - 0.01;
		if (colord == 1)
			sMaterialBody[3] = sMaterialBody[3] + 0.01;
		if (sMaterialBody[0] > 1)
			colora = 0;
		if (sMaterialBody[0] < 0)
			colora = 1;
		if (sMaterialBody[1] > 1)
			colorb = 0;
		if (sMaterialBody[1] < 0)
			colorb = 1;
		if (sMaterialBody[2] > 1)
			colorc = 0;
		if (sMaterialBody[2] < 0)
			colorc = 1;
		if (sMaterialBody[3] > 1)
			colord = 0;
		if (sMaterialBody[3] < 0)
			colord = 1;
	}
	printf("kDt:%lf\n", boid[1].kDt);
	for (int i = 0; i < kNumBoids; i++) {
		gx = 1;
		gy = 1;
		gz = 1;
		dgx = 1;// gx / numBoids - boid[i].x;
		dgy = 1;// gy / numBoids - boid[i].y;
		dgz = 1;// gz / numBoids - boid[i].z;
		d = sqrt(boid[i].vx*boid[i].vx + boid[i].vy*boid[i].vy + boid[i].vz*boid[i].vz);
		fx -= boid[i].kGain1*boid[i].x / d / d;
		fy -= boid[i].kGain1*boid[i].y / d / d;
		fz -= boid[i].kGain1*boid[i].z / d / d;
		fx += boid[i].kGain2*boid[i].vx / d / d;
		fy += boid[i].kGain2*boid[i].vy / d / d;
		fz += boid[i].kGain2*boid[i].vz / d / d;
		numBoids = rand() % 2;
		fx -= boid[i].kGainFv * boid[i].vx;
		fy -= boid[i].kGainFv * boid[i].vy;
		fz -= boid[i].kGainFv * boid[i].vz;
		if (boid[i].vx*boid[i].vx + boid[i].vy*boid[i].vy + boid[i].vz*boid[i].vz < .1) {
			fx += boid[i].vx;
			fy += boid[i].vy;
			fz += boid[i].vz;
		}
		boid[i].x += boid[i].vx * boid[i].kDt;
		boid[i].y += boid[i].vy * boid[i].kDt;
		boid[i].z += boid[i].vz * boid[i].kDt;
		boid[i].vx += fx / boid[i].kM * boid[i].kDt;
		boid[i].vy += fy / boid[i].kM * boid[i].kDt;
		boid[i].vz += fz / boid[i].kM * boid[i].kDt;
		if (boid[i].x > kRoomW / 2) boid[i].x -= kRoomW;
		if (boid[i].y > kRoomH / 2) boid[i].y -= kRoomH;
		if (boid[i].z > kRoomD / 2) boid[i].z -= kRoomD;
		if (boid[i].x < -kRoomW / 2) boid[i].x += kRoomW;
		if (boid[i].y < -kRoomH / 2) boid[i].y += kRoomH;
		if (boid[i].z < -kRoomD / 2) boid[i].z += kRoomD;
		if (boid[i].vx < -10 || boid[i].vy < -10 || boid[i].vz < -10)
		{
			boid[i].vx = 1; boid[i].vy = 1; boid[i].vz = 1;
		}

		counter += (int)(200 * sqrt(boid[i].vx * boid[i].vx + boid[i].vy * boid[i].vy + boid[i].vz * boid[i].vz) * boid[i].kDt);
		finAngle = sin(counter*.1) * 50;
		hire = cos(counter*.1) * 50;
		/*printf("x=%lf:y=%lf:z=%lf\n", boid[i].x, boid[i].y, boid[i].z);
		printf("fx=%lf:fy=%lf:fz=%lf\n", fx, fy, fz);
		printf("gx=%lf:gy=%lf:gz=%lf\n", gx, gy, gz);
		printf("vx=%lf:vy=%lf:vz=%lf\n", boid[i].vx, boid[i].vy, boid[i].vz);
		printf("kGain3=%lf:kGain2=%lf:kGain1=%lf\n", boid[i].kGain3, boid[i].kGain2, boid[i].kGain1);
		*/
	}
	glutPostRedisplay();
	glutTimerFunc(33, myTimer, 0);
}

void myInit(char *progname)
{
	glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA | GLUT_DEPTH);//ディスプレイモードの指定(ダブルバッファ|RGBA|デプス)
	glutInitWindowSize(500, 500);//ウィンドウサイズの指定(500,500)
	glutInitWindowPosition(0, 0);//表示座標
	glutCreateWindow(progname);//ウィンドウの生成
	glClearColor(.8, .8, .95, 1);//カラー情報を初期化
	glLoadIdentity();//変換処理の累積を消去
	GLUT_SET_MENU();//メニューの設定

	glLightModeli(GL_LIGHT_MODEL_TWO_SIDE, 1);//鏡面反射光成分を別々に補間
}

int main(int argc, char **argv)
{
	glutInit(&argc, argv);//初期化
	myInit(argv[0]);//初期化設定
	mySetLight();//ライト設定
	glutKeyboardFunc(myKeyboard);//キーボード・コールバックを設定
	glutMouseFunc(myMouseFunc);//マウス・コールバックを登録
	glutMotionFunc(myMotionFunc);//モーション・コールバックとパッシブ・モーション・コールバックを登録
	glutIdleFunc(myIdle);

	glutTimerFunc(33, myTimer, 0);//33ミリ秒後にコールされるタイマー・コールバックを登録
	glutReshapeFunc(myReshape);// リシェイプ・コールバックを指定
	glutDisplayFunc(myDisplay);//ディスプレイ・コールバックを指定

	srand48((int)time(NULL));//時間による乱数を生成
	for (int i = 0; i < kNumBoids; i++) {
		setPosition(i, drand48() * 4 - 2, drand48() * 4 - 2, drand48() * 4 - 2);
		setVelocity(i, drand48() * 2 - 1, drand48() * 2 - 1, drand48() * 2 - 1);
	}
	glutMainLoop();//イベント処理ループに入る
	return 0;
}





void setPosition(int i, double aX, double aY, double aZ)
{
	boid[i].x = aX;
	boid[i].y = aY;
	boid[i].z = aZ;
}

void setVelocity(int i, double aVx, double aVy, double aVz)
{
	boid[i].vx = aVx;
	boid[i].vy = aVy;
	boid[i].vz = aVz;
}

void draw(int i, double size)
{
	glMaterialfv(GL_FRONT_AND_BACK, GL_AMBIENT_AND_DIFFUSE, sMaterialBody);
	glScalef(.5*size, 1 * size, 1 * size);
	glRotatef(180, 1 * size, 0, 0);
	glutSolidCone(.1*size, .01*size, 10, 10);
	glRotatef(180, 1, 0, 0);
	glutSolidCone(.1*size, .1*size, 10, 10);
	glMaterialfv(GL_FRONT, GL_AMBIENT_AND_DIFFUSE, eyes_w);
	glTranslated(0.04*size, 0.05*size, 0.03*size);
	glutSolidSphere(0.02, 10, 10);
	glMaterialfv(GL_FRONT, GL_AMBIENT_AND_DIFFUSE, eyes_b);
	glTranslated(0, 0, 0.02*size);
	glutSolidSphere(0.01, 10, 10);
	glMaterialfv(GL_FRONT, GL_AMBIENT_AND_DIFFUSE, eyes_w);
	glTranslated(-0.08*size, 0, -0.02*size);
	glutSolidSphere(0.02, 10, 10);
	glMaterialfv(GL_FRONT, GL_AMBIENT_AND_DIFFUSE, eyes_b);
	glTranslated(0, 0, 0.02*size);
	glutSolidSphere(0.01, 10, 10);
	glMaterialfv(GL_FRONT, GL_AMBIENT_AND_DIFFUSE, sMaterialFin);
	glRotatef(finAngle, 0, 1, 0);  // move the fin
	glTranslatef(0.04*size, -0.05*size, -0.09*size);
	glScalef(0.25*size, 1, 1);
	glutSolidCone(0.04*size, .1*size, 10, 10);

}

void rotate(int i) {
	double p = atan2(boid[i].vy, sqrt(boid[i].vx*boid[i].vx + boid[i].vz*boid[i].vz));
	double q = atan2(boid[i].vx, boid[i].vz);
	glRotatef(q * 180 / M_PI, 0, 1, 0);
	glRotatef(-p * 180 / M_PI, 1, 0, 0);
}

void translate(int i)
{
	glTranslatef(boid[i].x, boid[i].y, boid[i].z);
}


