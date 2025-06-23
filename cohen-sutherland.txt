#include <graphics.h>
#include <conio.h>
#include <dos.h>   // For delay() function in Turbo C++
#include <math.h>  // For fabs() function

// Turbo C++ does not have bool type, so we define it manually
#define bool int
#define true 1
#define false 0

const int xmin = 70, ymin = 70, xmax = 430, ymax = 430;
const int TOP = 1, RIGHT = 2, BOTTOM = 4, LEFT = 8;

// Function to compute region code for clipping
int computeBitCode(int x, int y) {
    int code = 0;
    if (y > ymax) code |= TOP;
    if (y < ymin) code |= BOTTOM;
    if (x > xmax) code |= RIGHT;
    if (x < xmin) code |= LEFT;
    return code;
}

// Cohen-Sutherland Clipping Algorithm
void cohenSutherlandClip(int x1, int y1, int x2, int y2) {
    int code1 = computeBitCode(x1, y1);
    int code2 = computeBitCode(x2, y2);
    bool accept = false;

    while (true) {
        if ((code1 == 0) && (code2 == 0)) {
            accept = true;
            break;
        } else if ((code1 & code2) != 0) {
            break;  // Polygon is completely outside
        } else {
            int codeOut = code1 != 0 ? code1 : code2;
            int xIntersect, yIntersect;

            if (codeOut & TOP) {
                yIntersect = ymax;
                xIntersect = x1 + (yIntersect - y1) * (x2 - x1) / (y2 - y1);
            } else if (codeOut & RIGHT) {
                xIntersect = xmax;
                yIntersect = y1 + (xIntersect - x1) * (y2 - y1) / (x2 - x1);
            } else if (codeOut & BOTTOM) {
                yIntersect = ymin;
                xIntersect = x1 + (yIntersect - y1) * (x2 - x1) / (y2 - y1);
            } else {
                xIntersect = xmin;
                yIntersect = y1 + (xIntersect - x1) * (y2 - y1) / (x2 - x1);
            }

            if (codeOut == code1) {
                x1 = xIntersect;
                y1 = yIntersect;
                code1 = computeBitCode(x1, y1);
            } else {
                x2 = xIntersect;
                y2 = yIntersect;
                code2 = computeBitCode(x2, y2);
            }
        }
    }

    if (accept) {
        setcolor(BROWN);
        setlinestyle(SOLID_LINE, 0, 5);
        rectangle(x1, y1, x2, y2);
        setfillstyle(SOLID_FILL, BROWN);
        floodfill((x1 + x2) / 2, (y1 + y2) / 2, BROWN);
    }
}

// Function to animate the bouncing ball
void animateBall() {
    int ballX = 200, ballY = 200;  // Ball's initial position
    int dx = 5, dy = 3;            // Ball's movement speed
    static int polyX1 = 450, polyY1 = 300, polyX2 = 550, polyY2 = 320;  // Polygon position

    while (!kbhit()) {
        cleardevice();
        setcolor(RED);
        rectangle(xmin, ymin, xmax, ymax);

        ballX += dx;
        ballY += dy;

        // Bounce logic for the ball within the clipping window
        if (ballX < xmin + 10) {
            ballX = xmin + 10;
            dx = -dx;
        }
        if (ballX > xmax - 10) {
            ballX = xmax - 10;
            dx = -dx;
        }
        if (ballY < ymin + 10) {
            ballY = ymin + 10;
            dy = -dy;
        }
        if (ballY > ymax - 10) {
            ballY = ymax - 10;
            dy = -dy;
        }

        // Collision detection with the moving polygon (rectangle)
        if (ballY + 10 >= polyY1 && ballY - 10 <= polyY2 && ballX + 10 >= polyX1 && ballX - 10 <= polyX2) {
            if (fabs(dy) > fabs(dx)) {  // Use fabs() instead of abs()
                dx = -dx;  // Reflect horizontally
            } else {
                dy = -dy;  // Reflect vertically
            }
            ballX += dx * 0.5;
            ballY += dy * 0.5;
        }

        // Draw the ball
        setcolor(WHITE);
        circle(ballX, ballY, 10);
        setfillstyle(SOLID_FILL, RED);
        floodfill(ballX, ballY, WHITE);

        // Move and clip the polygon (obstacle)
        polyX1 -= 5;
        polyX2 -= 5;

        cohenSutherlandClip(polyX1, polyY1, polyX2, polyY2);

        // Reset polygon position if it goes off-screen
        if (polyX1 < xmin - 50) {
            polyX1 = 450;
            polyX2 = 550;
        }

        delay(50);  // Frame rate control
    }
}

int main() {
    int gd = DETECT, gm;
    initgraph(&gd, &gm, "C:\\TURBOC3\\BGI");  // Initialize graphics in Turbo C++

    animateBall();  // Call the animation function

    getch();  // Wait for user input
    closegraph();  // Close the graphics mode
    return 0;
}
