# CTIS164-Homework-2
Bilkent University CTIS 164 Second Homework

/*********
   CTIS164 - Template Source Program
----------
STUDENT : Nur Sude Var
SECTION : 1
HOMEWORK: A game that is occured by hiitting the target by a weapon
----------
PROBLEMS: In the {0,0} point weapon is appeared
----------
ADDITIONAL FEATURES:
-'W': Switches the weapon between Candy or Bomb
-Up and down keys makes faster or slower the birds that generated after the button is pressed.
-'R': Reset the speed of the birds.
-Weapon is moving on the x axis by right and left buttons
*********/

#ifdef _MSC_VER
#define _CRT_SECURE_NO_WARNINGS 1
#endif
#include <GL/glut.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>
#include <stdarg.h>

#define WINDOW_WIDTH  800
#define WINDOW_HEIGHT 800

#define TIMER_PERIOD  16 // Period for the timer.
#define TIMER_ON         1 // 0:disable timer, 1:enable timer

#define D2R 0.0174532
#define MAX_FIRE 30
#define MAX_BIRD 4
#define FIRE_RATE 10
#define CANDY 1
#define ARROW 2
int mode = CANDY;
/* Global Variables for Template File */
bool up = false, down = false, right = false, left = false, spacebar = false, status = false;
int  winWidth, winHeight; // current Window width and height
bool selection = false;
int fire_rate = 0;
int counter = 0;
typedef struct {
    float x, y; //Candy and Bomb's Axis
} axis_t;
typedef struct {
    axis_t pos; //Candy and Bomb's Position
    bool active;
    float angle;
} bomb_t;
typedef struct {
    int x;
    axis_t pos;   //Weapon's position
    float angle;
    float r;
}weaponn_t;
typedef struct {
    int r, g, b;
}color_t;
typedef struct {
    axis_t center;
    color_t color;
    float speed;
}birds_t;
bomb_t fire[MAX_FIRE];
weaponn_t weapon[MAX_FIRE]; //For the movement of weapon itself to the right and left
birds_t birds[MAX_BIRD] = { 0 }; //For the movement of birds
int count = 0;

//
// to draw circle, center at (x,y)
// radius r
//
void circle(int x, int y, int r)
{
#define PI 3.1415
    float angle;
    glBegin(GL_POLYGON);
    for (int i = 0; i < 100; i++)
    {
        angle = 2 * PI * i / 100;
        glVertex2f(x + r * cos(angle), y + r * sin(angle));
    }
    glEnd();
}

void circle_wire(int x, int y, int r)
{
#define PI 3.1415 
    float angle;

    glBegin(GL_LINE_LOOP);
    for (int i = 0; i < 100; i++)
    {
        angle = 2 * PI * i / 100;
        glVertex2f(x + r * cos(angle), y + r * sin(angle));
    }
    glEnd();
}

void print(int x, int y, const char* string, void* font)
{
    int len, i;

    glRasterPos2f(x, y);
    len = (int)strlen(string);
    for (i = 0; i < len; i++)
    {
        glutBitmapCharacter(font, string[i]);
    }
}

void vprint(int x, int y, void* font, const char* string, ...)
{
    va_list ap;
    va_start(ap, string);
    char str[1024];
    vsprintf_s(str, string, ap);
    va_end(ap);

    int len, i;
    glRasterPos2f(x, y);
    len = (int)strlen(str);
    for (i = 0; i < len; i++)
    {
        glutBitmapCharacter(font, str[i]);
    }
}


void vprint2(int x, int y, float size, const char* string, ...) {
    va_list ap;
    va_start(ap, string);
    char str[1024];
    vsprintf_s(str, string, ap);
    va_end(ap);
    glPushMatrix();
    glTranslatef(x, y, 0);
    glScalef(size, size, 1);

    int len, i;
    len = (int)strlen(str);
    for (i = 0; i < len; i++)
    {
        glutStrokeCharacter(GLUT_STROKE_ROMAN, str[i]);
    }
    glPopMatrix();
}

void BackgroundColor() {

    glColor3ub(65, 105, 225);
    glBegin(GL_QUADS);
    glVertex2f(-400, 400);
    glVertex2f(400, 400);
    glColor3ub(135, 206, 250);
    glVertex2f(400, -400);
    glVertex2f(-400, -400);
    glEnd();

}
void displayScreen()
{
    glColor3ub(72, 61, 139);
    glRectf(-400, 400, -390, -400);

    glColor3ub(72, 61, 139);
    glRectf(400, 400, 390, -400);

    glColor3ub(72, 61, 139);
    glRectf(-400, 390, 400, 400);

    glColor3ub(72, 61, 139);
    glRectf(-400, -390, 400, -400);

}
void displayWhiteScreen() {
    glColor3ub(255, 255, 255);
    glRectf(-800, -400, -400, 400);

    glColor3ub(255, 255, 255);
    glRectf(800, -400, 400, 400);
}
void displayBackground() {

    //Curtain
    glColor3ub(255, 192, 203);
    circle(-290, 400, 300);

    //Curtain
    glColor3ub(255, 192, 203);
    circle(290, 400, 300);

    //Curtain
    glColor3ub(219, 112, 147);
    circle(-400, 400, 300);

    //Curtain 
    glColor3ub(219, 112, 147);
    circle(400, 400, 300);

    //Curtain
    glColor3ub(255, 192, 203);
    glRectf(-400, -400, -300, 400);

    //Curtain
    glColor3ub(255, 192, 203);
    glRectf(300, -400, 400, 400);

    //Curtain
    glColor3ub(255, 0, 0);
    glRectf(-400, 90, -300, 100);

    //Curtain
    glColor3ub(255, 0, 0);
    glRectf(400, 90, 300, 100);
}

//void candyWeapon() {
//
//    //Candy
//    glColor3ub(255, 192, 203);
//    circle(0, -223, 17);
//    // Inside of candy
//    glColor3ub(255, 20, 147);
//    circle(0, -223, 10);
//
//
//    //The most inside of candy
//    glColor3ub(230, 230, 250);
//    circle(0, -223, 5);
//
//}

void arrowWeapon() {
    //Weapon Itself
    glColor3ub(0, 0, 0);
    circle(0, -223, 17);
}
void displayWeaponSelection() {

    //Left Selection Bar
    glColor3f(1, 1, 1);
    glRectf(weapon->pos.x - 80, -300, weapon->pos.x - 40, -340);

    //Right Selection Bar
    glColor3f(1, 1, 1);
    glRectf(weapon->pos.x + 40, -300, weapon->pos.x + 80, -340);

    //Left Side Candy Selection Button
    //Candy
    glColor3ub(255, 192, 203);
    circle(weapon->pos.x - 60, -320, 15);
    // Inside of candy
    glColor3ub(255, 20, 147);
    circle(weapon->pos.x - 60, -320, 10);

    //The most inside of candy
    glColor3ub(230, 230, 250);
    circle(weapon->pos.x - 60, -320, 5);

    //Weapon Itself
    glColor3ub(0, 0, 0);
    circle(weapon->pos.x + 60, -320, 15);

}
void displayWeapon() {
    //Tekerlek of weapon
    glColor3ub(239, 255, 128);
    circle(weapon->pos.x - 50, -380, 10);
    //Tekerlek of weapon
    glColor3ub(239, 255, 128);
    circle(weapon->pos.x + 50, -380, 10);

    //Body of weapon
    glColor3f(0.55, 0.610, 0.99);
    glRectf(weapon->pos.x - 100, -280, weapon->pos.x + 100, -375);

    //Weapon atış kısmı
    glColor3f(0.23, 0.610, 0.852);
    glRectf(weapon->pos.x - 20, -240, weapon->pos.x + 20, -280);

    displayWeaponSelection();

}
int TargetBirds() {
    int min = -(WINDOW_WIDTH / 2) - 80;
    for (int i = 0; i < MAX_BIRD; i++)
        if (birds[i].center.x < min) min = birds[i].center.x;

    return min;
}

void drawFire(bomb_t* f) {
    if (f->active) {
        for (int i = 0; i < MAX_FIRE; i++)
        {

            if (mode == CANDY) {
                //Candy
                glColor3ub(255, 192, 203);
                circle(weapon[i].pos.x, f[i].pos.y, 15);
                // Inside of candy  
                glColor3ub(255, 20, 147);
                circle(weapon[i].pos.x, f[i].pos.y, 10);
                //The most inside of candy
                glColor3ub(230, 230, 250);
                circle(weapon[i].pos.x, f[i].pos.y, 5);

            }
            else
            {
                glColor3ub(0, 0, 0);
                circle(weapon[i].pos.x, f[i].pos.y, 15);
            }


        }


    }
}
void displayBirds(birds_t* bird) {
    //Body of the bird
    glBegin(GL_POLYGON);
    glColor3ub(bird->color.r, bird->color.g, bird->color.b);
    glVertex2d(bird->center.x + 20, bird->center.y + 50);
    glVertex2d(bird->center.x + 20, bird->center.y + 130);
    glVertex2d(bird->center.x + 100, bird->center.y + 130);
    glVertex2d(bird->center.x + 100, bird->center.y + 50);
    glEnd();

    //Wing of the bird
    glBegin(GL_POLYGON);
    glColor3ub(223, 179, 255);
    glVertex2d(bird->center.x + 22, bird->center.y + 70);
    glVertex2d(bird->center.x + 22, bird->center.y + 110);
    glVertex2d(bird->center.x + 60, bird->center.y + 110);
    glVertex2d(bird->center.x + 60, bird->center.y + 70);
    glEnd();

    //Eye part  of the bird
    glBegin(GL_POLYGON);
    glColor3ub(255, 255, 255);
    glVertex2d(bird->center.x + 80, bird->center.y + 110);
    glVertex2d(bird->center.x + 80, bird->center.y + 120);
    glVertex2d(bird->center.x + 90, bird->center.y + 120);
    glVertex2d(bird->center.x + 90, bird->center.y + 110);
    glEnd();

    //Iris of the bird
    glColor3ub(1, 1, 1);
    circle(bird->center.x + 85, bird->center.y + 115, 3);

    //Mouth of the bird
    glBegin(GL_POLYGON);
    glColor3ub(255, 0, 0);
    glVertex2d(bird->center.x + 80, bird->center.y + 75);
    glVertex2d(bird->center.x + 80, bird->center.y + 85);
    glVertex2d(bird->center.x + 98, bird->center.y + 85);
    glVertex2d(bird->center.x + 98, bird->center.y + 75);
    glEnd();

    //Smile of the bird
    glBegin(GL_POLYGON);
    glColor3ub(255, 0, 0);
    glVertex2d(bird->center.x + 80, bird->center.y + 75);
    glVertex2d(bird->center.x + 80, bird->center.y + 95);
    glVertex2d(bird->center.x + 87, bird->center.y + 95);
    glVertex2d(bird->center.x + 87, bird->center.y + 75);
    glEnd();

    /*
    Code explanation:
    I let a couple of cm spaces between polygons because otherwise it looks like mouth and eye are outside of the bird
    */
}
void displayInfo() {
    glColor3ub(1, 1, 1);
    vprint(-389, 378, GLUT_BITMAP_8_BY_13, "NUR SUDE VAR");
    vprint(-389, 367, GLUT_BITMAP_8_BY_13, "21903502");
    vprint(-389, 356, GLUT_BITMAP_8_BY_13, "HOMEWORK 2");
    vprint(-389, 345, GLUT_BITMAP_8_BY_13, "Trick or Treat Game.");
    vprint(-389, 334, GLUT_BITMAP_8_BY_13, "Choose your weapon.A candy or a bomb.");
    vprint(-389, 323, GLUT_BITMAP_8_BY_13, "Then start to PLAY!");
    vprint(-390, -200, GLUT_BITMAP_8_BY_13, "press <up/down>");
    vprint(-390, -211, GLUT_BITMAP_8_BY_13, "for harder and easier version");
    vprint(-390, -222, GLUT_BITMAP_8_BY_13, "press <right/left");
    vprint(-390, -233, GLUT_BITMAP_8_BY_13, "for changing the location of weapon");

    vprint(weapon->pos.x - 60, -355, GLUT_BITMAP_TIMES_ROMAN_10, "Click W for switching the bomb");

}
//
// To display onto window using OpenGL commands
//
void display() {
    //
    // clear window to black
    //
    glClearColor(65, 105, 225, 1);
    glClear(GL_COLOR_BUFFER_BIT);
    BackgroundColor();
    displayBackground();
    displayWeapon();
    displayInfo();
    for (int i = 0; i < MAX_BIRD; i++)
        displayBirds(birds + i);

    //for (int i = 0; i < MAX_FIRE; i++)
    drawFire(fire);
    displayWhiteScreen();
    displayScreen();
    glutSwapBuffers();

}
int findAvailableFire() {
    for (int i = 0; i < MAX_FIRE; i++)
        if (fire[i].active == false) return i;
    return -1;
}
void resetTarget(birds_t* bird) {
    float xPos = rand() % (WINDOW_HEIGHT / 2) - WINDOW_HEIGHT;
    float lowest = TargetBirds();
    float yPos = (rand() % (350 - (100) + 1)) + (0);
    bird->center.x = fmin(xPos, lowest) - 150;
    bird->center.y = yPos;
    bird->color.r = rand() % 256;
    bird->color.g = rand() % 256;
    bird->color.b = rand() % 256;
    bird->speed = 10.0;
}
//
// key function for ASCII charachters like ESC, a,b,c..,A,B,..Z
//
void onKeyDown(unsigned char key, int x, int y)
{
    // exit when ESC is pressed.
    if (key == 27)
        exit(0);
    switch (key) {
    case ' ': spacebar = true;
        // firing = true;
        break;
    case 'W':
    case 'w': if (mode == CANDY)
        mode++;
            else if (mode == ARROW)
        mode--;
        break;
    case 'R':
    case 'r': if (status = true)
        for (int i = 0; i < MAX_BIRD; i++)
            birds[i].speed = 10.0;
        break;
    }

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

void onKeyUp(unsigned char key, int x, int y)
{
    // exit when ESC is pressed.
    if (key == 27)
        exit(0);
    if (key == ' ')
        spacebar = false;
    //5firing = false;


// to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// Special Key like GLUT_KEY_F1, F2, F3,...
// Arrow Keys, GLUT_KEY_UP, GLUT_KEY_DOWN, GLUT_KEY_RIGHT, GLUT_KEY_RIGHT
//
void onSpecialKeyDown(int key, int x, int y)
{
    // Write your codes here.
    switch (key) {
    case GLUT_KEY_UP: up = true; break;
    case GLUT_KEY_DOWN: down = true; break;
    case GLUT_KEY_LEFT: left = true; break;
    case GLUT_KEY_RIGHT: right = true; break;
    }

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// Special Key like GLUT_KEY_F1, F2, F3,...
// Arrow Keys, GLUT_KEY_UP, GLUT_KEY_DOWN, GLUT_KEY_RIGHT, GLUT_KEY_RIGHT
//
void onSpecialKeyUp(int key, int x, int y)
{
    // Write your codes here.
    switch (key) {
    case GLUT_KEY_UP: up = false; break;
    case GLUT_KEY_DOWN: down = false; break;
    case GLUT_KEY_LEFT: left = false; break;
    case GLUT_KEY_RIGHT: right = false; break;
    }

    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// When a click occurs in the window,
// It provides which button
// buttons : GLUT_LEFT_BUTTON , GLUT_RIGHT_BUTTON
// states  : GLUT_UP , GLUT_DOWN
// x, y is the coordinate of the point that mouse clicked.
//
void onClick(int button, int stat, int x, int y)
{
    // Write your codes here.



    // to refresh the window it calls display() function
    glutPostRedisplay();
}

//
// This function is called when the window size changes.
// w : is the new width of the window in pixels.
// h : is the new height of the window in pixels.
//
void onResize(int w, int h)
{
    winWidth = w;
    winHeight = h;
    glViewport(0, 0, w, h);
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    glOrtho(-w / 2, w / 2, -h / 2, h / 2, -1, 1);
    glMatrixMode(GL_MODELVIEW);
    glLoadIdentity();
    display(); // refresh window.
}

void onMoveDown(int x, int y) {
    // Write your codes here.



    // to refresh the window it calls display() function   
    glutPostRedisplay();
}

// GLUT to OpenGL coordinate conversion:
//   x2 = x1 - winWidth / 2
//   y2 = winHeight / 2 - y1
void onMove(int x, int y) {
    // Write your codes here.



    // to refresh the window it calls display() function
    glutPostRedisplay();
}

bool hit(bomb_t fire, birds_t bird) {
    count++;
    return(((fire.pos.y + 15) == bird.center.y) && ((fire.pos.x < bird.center.x + 100) && (fire.pos.x > bird.center.x)));
}

bool doesHit(float x1, float y1, float x2, float y2, int width, int height) {
    return x1 >= x2 && x1 <= x2 + width && y1 >= y2 && y1 <= y2 + height;
}

#if TIMER_ON == 1
void onTimer(int v) {

    glutTimerFunc(TIMER_PERIOD, onTimer, 0);
    // Write your codes here.
    for (int i = 0; i < MAX_BIRD; i++)
    {
        if (up) birds[i].speed += 2.0;
        if (down) birds[i].speed -= 2.0;
        if (birds[i].speed <= 0) birds[i].speed = 2.0, status = true;
        if (birds[i].speed >= 50) birds[i].speed = 50.0, status = true;

    }

    if (right) weapon->pos.x += 5;
    if (left) weapon->pos.x -= 5;
    if (weapon->pos.x + 105 >= winWidth / 2) weapon->pos.x -= 15;
    if (weapon->pos.x - 105 <= -winWidth / 2)weapon->pos.x += 15;

    if (weapon->pos.x >= winWidth / 2)
        weapon->pos.x = -winWidth / 20;

    if (spacebar && fire->active == false) {
        fire->pos = { 0,-230 };
        fire->active = true;
        //fire_rate = FIRE_RATE;
    }
    if (fire->active && mode == CANDY) {
        fire->pos.y += 5;

        for (int i = 0; i < MAX_BIRD; i++) {
            birds_t* bird = birds + i;
            if (doesHit(weapon->pos.x, fire->pos.y, bird->center.x + 20, bird->center.y + 50, 80, 80)) resetTarget(birds + i);
        }

        if (fire->pos.x > 390 || fire->pos.x < -390 || fire->pos.y>390 || fire->pos.y < -390) {
            fire->active = false;
        }
    }
    if (fire->active && mode == ARROW) {
        fire->pos.y += 5;

        for (int i = 0; i < MAX_BIRD; i++) {
            birds_t* bird = birds + i;
            if (doesHit(weapon->pos.x, fire->pos.y, bird->center.x + 20, bird->center.y + 50, 80, 80)) resetTarget(birds + i);
        }

        if (fire->pos.x > 390 || fire->pos.x < -390 || fire->pos.y>390 || fire->pos.y < -390) {
            fire->active = false;
        }
    }

    if (fire->active) {
        fire->pos.x += 10;
        fire->pos.y += 10;

        if (fire->pos.x > 370 || fire->pos.x < -370 || fire->pos.y>370 || fire->pos.y < -370) {
            fire->active = false;
        }

    }
    // move target from left to right
    for (int i = 0; i < MAX_BIRD; i++) {
        birds_t* bird = birds + i;
        bird->center.x += bird->speed;
        if (bird->center.x > 300) {
            resetTarget(bird);
        }
    }

    // to refresh the window it calls display() function
    glutPostRedisplay(); // display()

}
#endif

void Init() {

    // Smoothing shapes
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    for (int i = 0; i < MAX_BIRD; i++)
        resetTarget(birds + i);
}

void main(int argc, char* argv[]) {
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE);
    glutInitWindowSize(WINDOW_WIDTH, WINDOW_HEIGHT);
    //glutInitWindowPosition(100, 100);
    glutCreateWindow("Template File");

    glutDisplayFunc(display);
    glutReshapeFunc(onResize);

    //
    // keyboard registration
    //
    glutKeyboardFunc(onKeyDown);
    glutSpecialFunc(onSpecialKeyDown);

    glutKeyboardUpFunc(onKeyUp);
    glutSpecialUpFunc(onSpecialKeyUp);

    //
    // mouse registration
    //
    glutMouseFunc(onClick);
    glutMotionFunc(onMoveDown);
    glutPassiveMotionFunc(onMove);

#if  TIMER_ON == 1
    // timer event
    glutTimerFunc(TIMER_PERIOD, onTimer, 0);
#endif

    Init();

    glutMainLoop();
}
