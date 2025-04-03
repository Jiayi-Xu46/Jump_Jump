#include <stdlib.h>   // for abs() if needed
#include <stdio.h>   // For sprintf
#include <string.h>  // For strlen
#include <math.h>

#define AUDIO_BASE 0xFF203040
#define TIMER_BASE 0xFF202000

#define NUM_DROPS 200
typedef struct {
    float x;
    float y;
    float vx;
    float vy;
} BloodDrop;

// get pointer to audio fifo
struct audio_t {
  unsigned int control;
  unsigned char rarc;
  unsigned char ralc;
  unsigned char wsrc;
  unsigned char wslc;
  unsigned int ldata;
  unsigned int rdata;
};

volatile struct audio_t* const audiop = ((struct audio_t*)AUDIO_BASE);

// Use a macro to combine the r, g and b components.
#define RGB16(r, g, b) (((r & 0x1F) << 11) | ((g & 0x3F) << 5) | (b & 0x1F))
#define CHAR_WIDTH 8
#define CHAR_HEIGHT 8
#define SCREEN_WIDTH 320
#define SCREEN_HEIGHT 240
#define CHARACTER_SIZE 20
#define MOVE_STEP 5
#define THRESHOLD 200000000

// Properly align buffers in SDRAM
short int Buffer1[SCREEN_HEIGHT][512];  // Front buffer
short int Buffer2[SCREEN_HEIGHT][512];  // Back buffer
// Game Settings
int pixel_buffer_start; // global variable
int groundHeight = 120;
int gameOn = 0; // Is main game loop alive.
int enemyX[6];
int enemyY[6];
int enemyLenght = 20;
int enemyHeight = 20;
int heroRadius = 8;
int heroX = 160;
int heroY = 130;
short int BLACK = RGB16(0, 0, 0);
short int GREEN = RGB16(0, 63, 0);
short int BLUE = RGB16(0, 0, 31);
short int RED = RGB16(31, 0, 0);
short int PINK = RGB16(31, 20, 31);
short int GOLD = RGB16(31, 27, 0);
short int CYAN   = RGB16(0, 63, 31);  // Full green and blue create cyan.
short int ORANGE = RGB16(31, 41, 0);   // A mix of red and a moderate amount of green produces orange.
short int PURPLE = RGB16(25, 0, 25);   // A balanced mix of red and blue gives a purple hue.
int hero_velocity = 0;    // Vertical speed of the hero
int enemy_up_velocity = 0;
int gravity = 2;          // Acceleration due to gravity
int enemy_gravity = 1;
int safeDistance = 5;
int score = 0;
int gameOver = 1;
int difficulty = 1;
int highScore = 0;
int is_shifted = 0;
int is_switched = 0;
int is_mutated = 0;
int Y_coordinate[4];
int side = 0;
int playAni = 0;
volatile int * ps2_ptr = (volatile int *)0xFF200100;
char key_held = 0;
int tut0_progress = 0;
int tut1_progress = 0;
int tut2_progress = 0;
int tut3_progress = 0;

int PS2_data, RAVLID;
char byte1=0, byte2=0, byte3=0;

char key=0;
int move =0;


// Function prototypes
void clear_screen();
void draw_horizontal_line(int row, short int color);
void plot_pixel(int x, int y, short int line_color);
void draw_box(int locationX, int locationY, int length, int height, short int color);
void draw_slime(int x, int y, int length, int height, short int color);
void wait_for_vsync();
int update_enemy_positions(int);
void update_hero_position(char, int);
void draw_game_over_screen();
void draw_circle(int x_center, int y_center, int radius, short int color);
void draw_string(int x, int y, const char* str, short int color);
void check_collision(int enemyX[], int enemyY[], int heroX, int heroY);
void check_revive(char, int);
void animate_ball_crack(int centerX, int centerY, int radius);
void animate_blood_splash(int startX, int startY);
void animate_blood_drops(int centerX, int centerY);
const short int hero[CHARACTER_SIZE][CHARACTER_SIZE];
const short int enemy[CHARACTER_SIZE][CHARACTER_SIZE];
const short int background[SCREEN_HEIGHT][SCREEN_WIDTH];
const short int slime[CHARACTER_SIZE][CHARACTER_SIZE];
// Read the starting address of the pixel buffer from the pixel controller
volatile int * pixel_ctrl_ptr = (int *)0xFF203020;
void animate_ball_crack_new(int centerX, int centerY, int radius);
void animate_revive(int centerX, int centerY, int finalRadius);
#define NUM_PARTICLES 200
// Define YELLOW as full red and green with no blue.
short int YELLOW = RGB16(31, 63, 0);
void check_start(char, int);
int isTut = 1;
int tut = 0;

typedef struct {
    float x;
    float y;
    float vx;
    float vy;
} Particle;

// NEW GLOBAL VARIABLES FOR NAME & LEADERBOARD
char playerName[20] = "";  // Buffer for the player's name
typedef struct {
    char name[20];
    int score;
} LeaderboardEntry;
#define MAX_LEADERBOARD_ENTRIES 5
LeaderboardEntry leaderboard[MAX_LEADERBOARD_ENTRIES];
int leaderboardCount = 0;
int leaderboardUpdated = 0;  // Flag to update leaderboard once per death

// ------------------------------------------------------------------
// NEW FUNCTIONS FOR NAME INPUT & LEADERBOARD
// ------------------------------------------------------------------
void clear_both_buffers() {
    // Set front buffer to Buffer1
    *(pixel_ctrl_ptr + 1) = (int)Buffer1;
    wait_for_vsync();
    pixel_buffer_start = *pixel_ctrl_ptr;  // Set front buffer pointer
    clear_screen(Buffer1);

    // Set back buffer to Buffer2
    *(pixel_ctrl_ptr + 1) = (int)Buffer2;
    pixel_buffer_start = *(pixel_ctrl_ptr + 1);
    clear_screen(Buffer2);
}


// A simple (partial) mapping from PS/2 scancodes to ASCII.
// (Extend this mapping as needed for full alphanumeric support.)
char ps2_to_ascii(char scancode) {
    switch(scancode) {
         case 0x1C: return 'a';
         case 0x32: return 'b';
         case 0x21: return 'c';
         case 0x23: return 'd';
         case 0x24: return 'e';
         case 0x2B: return 'f';
         case 0x34: return 'g';
         case 0x33: return 'h';
         case 0x43: return 'i';
         case 0x3B: return 'j';
         case 0x42: return 'k';
         case 0x4B: return 'l';
         case 0x3A: return 'm';
         case 0x31: return 'n';
         case 0x44: return 'o';
         case 0x4D: return 'p';
         case 0x15: return 'q';
         case 0x2D: return 'r';
         case 0x1B: return 's';
         case 0x2C: return 't';
         case 0x3C: return 'u';
         case 0x2A: return 'v';
         case 0x1D: return 'w';
         case 0x22: return 'x';
         case 0x35: return 'y';
         case 0x1A: return 'z';
         // Digits:
         case 0x45: return '0';
         case 0x16: return '1';
         case 0x1E: return '2';
         case 0x26: return '3';
         case 0x25: return '4';
         case 0x2E: return '5';
         case 0x36: return '6';
         case 0x3D: return '7';
         case 0x3E: return '8';
         case 0x46: return '9';
         // Special keys: assume 0x29 is Enter and 0x66 is Backspace.
         case 0x29: return '\n'; // Enter key
         case 0x66: return '\b'; // Backspace key
         default: return 0;
    }
}

// Draws the name input screen
void draw_name_input_screen() {
    
    char prompt[] = "Enter Your Name:";
    int textWidth = strlen(prompt) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 80;
    draw_string(xPos, yPos, prompt, GREEN);
    
    // Display the current name input
    int nameWidth = strlen(playerName) * CHAR_WIDTH;
    int xName = (320 - nameWidth) / 2;
    int yName = yPos + 20;
    draw_string(xName, yName, playerName, YELLOW);
    
    char instr[] = "Press Enter to start";
    int instrWidth = strlen(instr) * CHAR_WIDTH;
    int xInstr = (320 - instrWidth) / 2;
    int yInstr = yName + 20;
    draw_string(xInstr, yInstr, instr, BLUE);
}

void tut0_prompt() {
    char prompt[] = "Press KEY2 or 'W' to jump.";
    int textWidth = strlen(prompt) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 30;
    draw_string(xPos, yPos, prompt, PURPLE);

    char instr[] = "Do 3 jumps to finish this tutorial.";
    int instrWidth = strlen(instr) * CHAR_WIDTH;
    int xInstr = (320 - instrWidth) / 2;
    int yInstr = 50;
    draw_string(xInstr, yInstr, instr, BLUE);
}

void check_tut0(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    if(move){
        if ((key_val & 0x2) != 0) {

            char instr[] = "Nice!";
            int instrWidth = strlen(instr) * CHAR_WIDTH;
            int xInstr = (320 - instrWidth) / 2;
            int yInstr = 200;
            draw_string(xInstr, yInstr, instr, GREEN);

            tut0_progress += 1;
            if (tut0_progress >= 30) {
                tut = 1;
                clear_screen();
            }
        }        
    }
}

// (key_val & 0x8) != 0 && (key_val & 0x4) != 0 && (key_val & 0x2) != 0 && (key_val & 0x1) != 0 

void check_tut1(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    if(move){
        if ((key_val & 0x4) != 0 && (key_val & 0x2) != 0) {

            char instr[] = "Awsome!";
            int instrWidth = strlen(instr) * CHAR_WIDTH;
            int xInstr = (320 - instrWidth) / 2;
            int yInstr = 200;
            draw_string(xInstr, yInstr, instr, GREEN);

            tut1_progress += 1;
            if (tut1_progress >= 30) {
                tut = 2;
                clear_screen();
            }
        }        
    }
}

void check_tut2(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    if(move){
        if ((key_val & 0x8) != 0 || (key_val & 0x4) != 0) {

            char instr[] = "Niu Bi!";
            int instrWidth = strlen(instr) * CHAR_WIDTH;
            int xInstr = (320 - instrWidth) / 2;
            int yInstr = 200;
            draw_string(xInstr, yInstr, instr, GREEN);

            tut2_progress += 1;
            if (tut2_progress >= 30) {
                tut = 3;
                clear_screen();
            }
        }        
    }
}

void check_tut3(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    if(move){
        if ((key_val & 0x2) != 0 || (key_val & 0x1) != 0) {

            char instr[] = "Superbe!";
            int instrWidth = strlen(instr) * CHAR_WIDTH;
            int xInstr = (320 - instrWidth) / 2;
            int yInstr = 200;
            draw_string(xInstr, yInstr, instr, GREEN);

            tut3_progress += 1;
            if (tut3_progress >= 30) {
                isTut = 0;
            }
        }        
    }
}

void tut1_prompt() {
    char prompt[] = "Press KEY2 + KEY3 to jump right";
    int textWidth = strlen(prompt) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 30;
    draw_string(xPos, yPos, prompt, PURPLE);
}

void tut2_prompt() {
    char prompt[] = "Use KEY3 and KEY4 to move left and right";
    int textWidth = strlen(prompt) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 30;
    draw_string(xPos, yPos, prompt, PURPLE);
}

void tut3_prompt() {
    char prompt[] = "Press KEY2 and KEY1 to switch sides";
    int textWidth = strlen(prompt) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 30;
    draw_string(xPos, yPos, prompt, PURPLE);
}


// Handles PS/2 input for name entry.
// When Enter is detected (scancode 0x29), the state changes to gameplay.
void check_name_input(char key, int move) {
    if (move) {
        delay(1);
        char ascii = ps2_to_ascii(key);
        if (ascii) {
            if (ascii == '\n') {  // Enter key pressed
                // Transition to gameplay state
                animate_revive(heroX, heroY, heroRadius);
                gameOn = 1;
            } else if (ascii == '\b') {  // Backspace
                int len = strlen(playerName);
                if (len > 0) {
                    playerName[len-1] = '\0';
                }
            } else {
                // Append character if there is room
                if (strlen(playerName) < 19) {
                    int len = strlen(playerName);
                    playerName[len] = ascii;
                    playerName[len+1] = '\0';
                    
                }
            }
        }
    }
}

// Updates the leaderboard with the current player's score and name.
// If the leaderboard is full, it replaces the entry with the lowest score if necessary.
void update_leaderboard() {
    if (score >= 0) {
        int found = 0;
        // Check if the player is already in the leaderboard
        for (int i = 0; i < leaderboardCount; i++) {
            if (strcmp(playerName, leaderboard[i].name) == 0) {
                leaderboard[i].score = score;
                found = 1;
                break;  // Assuming one entry per player
            }
        }
        
        // If not found, add a new entry
        if (!found) {
            if (leaderboardCount < MAX_LEADERBOARD_ENTRIES) {
                leaderboard[leaderboardCount].score = score;
                strcpy(leaderboard[leaderboardCount].name, playerName);
                leaderboardCount++;
            } else {
                // Find the entry with the lowest score
                int minIndex = 0;
                for (int i = 1; i < MAX_LEADERBOARD_ENTRIES; i++) {
                    if (leaderboard[i].score < leaderboard[minIndex].score)
                        minIndex = i;
                }
                if (score > leaderboard[minIndex].score) {
                    leaderboard[minIndex].score = score;
                    strcpy(leaderboard[minIndex].name, playerName);
                }
            }
        }
        
        // Simple sort: descending order (highest score first)
        for (int i = 0; i < leaderboardCount - 1; i++) {
            for (int j = i + 1; j < leaderboardCount; j++) {
                if (leaderboard[j].score > leaderboard[i].score) {
                    LeaderboardEntry temp = leaderboard[i];
                    leaderboard[i] = leaderboard[j];
                    leaderboard[j] = temp;
                }
            }
        }
    }
}

// Draws the game over screen along with the leaderboard.
void draw_game_over_screen_with_leaderboard(void) {
    clear_screen();
    char gameOverMsg[] = "GAME OVER";
    int textWidth = strlen(gameOverMsg) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 50;
    draw_string(xPos, yPos, gameOverMsg, RED);
    
    // Display leaderboard title
    char lbTitle[] = "LEADERBOARD";
    int lbTitleWidth = strlen(lbTitle) * CHAR_WIDTH;
    int xLbTitle = (320 - lbTitleWidth) / 2;
    int yLbTitle = yPos + 30;
    draw_string(xLbTitle, yLbTitle, lbTitle, GOLD);
    
    // Display each leaderboard entry
    for (int i = 0; i < leaderboardCount; i++) {
         char entry[40];
         sprintf(entry, "%d. %s - %d", i + 1, leaderboard[i].name, leaderboard[i].score);
         int entryWidth = strlen(entry) * CHAR_WIDTH;
         int xEntry = (320 - entryWidth) / 2;
         int yEntry = yLbTitle + 20 + i * 10;
         draw_string(xEntry, yEntry, entry, GREEN);
    }
    
    char instructions[] = "Press all keys to revive";
    int instWidth = strlen(instructions) * CHAR_WIDTH;
    int xInst = (320 - instWidth) / 2;
    int yInst = 200;
    draw_string(xInst, yInst, instructions, GREEN);
}


void animate_reverse_blood_spring(int targetX, int targetY) {
    int numFrames = 50;  // Total number of animation frames.
    Particle particles[NUM_PARTICLES];
    int R = 200;  // Maximum initial offset from the target point.
    
    // Initialize particles with random offsets around the target,
    // and compute per-frame velocities to converge to (targetX, targetY).
    for (int i = 0; i < NUM_PARTICLES; i++) {
        int dx = (rand() % (2 * R + 1)) - R;  // Random value in [-R, R]
        int dy = (rand() % (2 * R + 1)) - R;  // Random value in [-R, R]
        particles[i].x = targetX + dx;
        particles[i].y = targetY + dy;
        particles[i].vx = - (float)dx / numFrames;
        particles[i].vy = - (float)dy / numFrames;
    }
    
    // Animation loop: update and draw particles on each frame.
    for (int frame = 0; frame < numFrames; frame++) {
        // Clear the back buffer before drawing this frame.
        clear_screen();
        
        // Update each particle and plot it as a yellow pixel.
        for (int i = 0; i < NUM_PARTICLES; i++) {
            particles[i].x += particles[i].vx;
            particles[i].y += particles[i].vy;
            int px = (int) particles[i].x;
            int py = (int) particles[i].y;
            if (px >= 0 && px < SCREEN_WIDTH && py >= 0 && py < SCREEN_HEIGHT) {
                plot_pixel(px, py, YELLOW);
            }
        }
        
        // Optionally, you could emphasize the target point:
        // draw_circle(targetX, targetY, 2, YELLOW);
        
        // Swap buffers to display the current frame.
        wait_for_vsync();
        volatile int *pixel_ctrl_ptr = (int *)0xFF203020;
        pixel_buffer_start = *(pixel_ctrl_ptr + 1);
        
        // Delay briefly for smoother animation.
        delay(1);
    }
}

void increase_xp(int targetX, int targetY) {
    int numFrames = 50;  // Total number of animation frames.
    Particle particles[NUM_PARTICLES];
    int R = 20;  // Maximum initial offset from the target point.
    
    // Initialize particles with random offsets around the target,
    // and compute per-frame velocities to converge to (targetX, targetY).
    for (int i = 0; i < NUM_PARTICLES; i++) {
        int dx = (rand() % (2 * R + 1)) - R;  // Random value in [-R, R]
        int dy = (rand() % (2 * R + 1)) - R;  // Random value in [-R, R]
        particles[i].x = targetX + dx;
        particles[i].y = targetY + dy;
        particles[i].vx = - (float)dx / numFrames;
        particles[i].vy = - (float)dy / numFrames;
    }
    
    // Animation loop: update and draw particles on each frame.
    for (int frame = 0; frame < numFrames; frame++) {
        
        // Update each particle and plot it as a yellow pixel.
        for (int i = 0; i < NUM_PARTICLES; i++) {
            particles[i].x += particles[i].vx;
            particles[i].y += particles[i].vy;
            int px = (int) particles[i].x;
            int py = (int) particles[i].y;
            if (px >= 0 && px < SCREEN_WIDTH && py >= 0 && py < SCREEN_HEIGHT) {
                plot_pixel(px, py, GREEN);
            }
        }
        
        // Optionally, you could emphasize the target point:
        // draw_circle(targetX, targetY, 2, YELLOW);
        
        // Swap buffers to display the current frame.
        wait_for_vsync();
        volatile int *pixel_ctrl_ptr = (int *)0xFF203020;
        pixel_buffer_start = *(pixel_ctrl_ptr + 1);

        // Delay briefly for smoother animation.
        delay(1);
    }
}

    
int main(void) {
    *ps2_ptr=0xff;
    int left;
    int right;
    int amplitude = 0;
    int max_amplitude = 0;

    // Game initialization.



    // Set front buffer to Buffer1
    *(pixel_ctrl_ptr + 1) = (int)Buffer1;
    wait_for_vsync();
    pixel_buffer_start = *pixel_ctrl_ptr;  // Set front buffer pointer
    clear_screen(Buffer1);

    // Set back buffer to Buffer2
    *(pixel_ctrl_ptr + 1) = (int)Buffer2;
    pixel_buffer_start = *(pixel_ctrl_ptr + 1);
    clear_screen(Buffer2);



    int enemyAltitiude = groundHeight - enemyHeight;
    heroY = groundHeight - heroRadius + 20;

    clear_screen();
   
    // Initialize enemy position
    enemyX[0] = 320;
    enemyX[1] = 400;
    enemyX[2] = 480;

    // Dungeon Enemies
    enemyX[3] = 0;
    enemyX[4] = -80;
    enemyX[5] = -160;

    // Four different places the blocks might spawn on
    Y_coordinate[0] = groundHeight - 2* enemyHeight;

    Y_coordinate[1] = groundHeight - enemyHeight;

    Y_coordinate[2] = groundHeight + 1;

    Y_coordinate[3] = groundHeight + enemyHeight;


    // Setting initial enemy y-location
    enemyY[0] = groundHeight - enemyHeight;
    enemyY[1] = groundHeight - enemyHeight;
    enemyY[2] = groundHeight - enemyHeight;

    for (int i = 3; i < 6; i++) {
        enemyY[i] = Y_coordinate[2];
    }


    // Draw background.
    draw_horizontal_line(groundHeight, RGB16(0, 25, 0));




    while (isTut) {
        PS2_data = *ps2_ptr;
        RAVLID = PS2_data&0X8000;

        if(RAVLID){
            byte1=byte2;
            byte2=byte3;
            byte3=PS2_data&0xff;
            if(byte2==0xf0){
                if(byte3==key_held){
                    key_held=0;
                    key=byte3;
                    move =0;
                }
            }else{
                key_held=byte3;
                key=key_held;
                move=1;
            }
        }

        switch(tut){

            case 0:
                draw_background();
                clear_hero(heroX, heroY);

                check_tut0(key, move);
                update_hero_position(key, move);
                draw_circle(heroX, heroY, heroRadius, PINK);
                draw_horizontal_line(groundHeight, RGB16(0, 25, 0));
                tut0_prompt();
                
                
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;

            case 1:
                
                draw_background();
                clear_hero(heroX, heroY);

                check_tut1(key, move);
                update_hero_position(key, move);
                draw_circle(heroX, heroY, heroRadius, PINK);
                draw_horizontal_line(groundHeight, RGB16(0, 25, 0));
                tut1_prompt();
                
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;

            case 2:
                
                draw_background();
                clear_hero(heroX, heroY);

                check_tut2(key, move);
                update_hero_position(key, move);
                draw_circle(heroX, heroY, heroRadius, PINK);
                draw_horizontal_line(groundHeight, RGB16(0, 25, 0));
                tut2_prompt();
                
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;


            case 3:
                
                draw_background();
                clear_hero(heroX, heroY);

                check_tut3(key, move);
                update_hero_position(key, move);
                draw_circle(heroX, heroY, heroRadius, PINK);
                draw_horizontal_line(groundHeight, RGB16(0, 25, 0));
                tut3_prompt();
                
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;   
        }
    }

    clear_screen();

    while (1) {
        PS2_data = *ps2_ptr;
        RAVLID = PS2_data&0X8000;

        if(RAVLID){
            byte1=byte2;
            byte2=byte3;
            byte3=PS2_data&0xff;
            if(byte2==0xf0){
                if(byte3==key_held){
                    key_held=0;
                    key=byte3;
                    move =0;
                }
            }else{
                key_held=byte3;
                key=key_held;
                move=1;
            }
        }

        switch(gameOn) {
            case 0:  // Name Input State
                clear_screen();
                delay(1);
                draw_name_input_screen();
                check_name_input(key, move);
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;

            case 1:
                draw_background();  // Draw background once

                // Erase prev. drawings.
                for (int i = 0; i < 6; i++) {
                    clear_enemy(enemyX[i] + 8, enemyY[i]);
                }
                clear_hero(heroX, heroY);


                if(audiop->rarc>0){
                    while(audiop->rarc>0){
                        int left = audiop->ldata;
                        int right = audiop->rdata;
                        amplitude = (left<0)? -left:left;
                        if(amplitude>max_amplitude){
                            max_amplitude=amplitude;
                        }
                    }
                }

                // Update Enemy Positions
                max_amplitude = update_enemy_positions(max_amplitude);

                //check_mutation();
                update_hero_position(key, move);

                // Draw enemies.
                draw_box(enemyX[0] + 8, enemyY[0], enemyLenght, enemyHeight, BLUE);
                draw_box(enemyX[1]+ 8, enemyY[1], enemyLenght, enemyHeight, RED);
                draw_box(enemyX[2]+ 8, enemyY[2], enemyLenght, enemyHeight, GOLD);

                draw_slime(enemyX[3]+ 8, enemyY[3], enemyLenght, enemyHeight, CYAN);
                draw_slime(enemyX[4]+ 8, enemyY[4], enemyLenght, enemyHeight, ORANGE);
                draw_slime(enemyX[5]+ 8, enemyY[5], enemyLenght, enemyHeight, PURPLE);

                draw_circle(heroX, heroY, heroRadius, PINK);



                if(score > highScore) {
                    highScore = score;
                    clear_score_left();
                }

                char scoreStr[16];
                sprintf(scoreStr, "Score: %d", score);

                // Calculate x coordinate so the string is right-aligned. (Assume screen width = 320.)
                int textWidth = strlen(scoreStr) * CHAR_WIDTH;
                int xPos = 320 - textWidth; // Top right corner.
                draw_string(xPos, 0, scoreStr, GREEN);

                char scoreStrMax[16];
                sprintf(scoreStrMax, "History Max: %d", highScore);

                // Calculate x coordinate so the string is right-aligned. (Assume screen width = 320.)
                int textWidthMax = strlen(scoreStrMax) * CHAR_WIDTH;
                int xPosMax = 30; // Top left corner.
                
                draw_string(xPosMax, 0, scoreStrMax, BLUE);

                // Check collision
                check_collision(enemyX, enemyY, heroX, heroY);
                // Draw background.
                draw_horizontal_line(groundHeight, RGB16(0, 25, 0));

                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);  // Swap buffer                    

            break;

            
            case 2:  // Death Screen with Leaderboard
                // On first entry into this state, update the leaderboard.
                if (!leaderboardUpdated) {
                    update_leaderboard();
                    leaderboardUpdated = 1;
                }
                draw_game_over_screen_with_leaderboard();
                check_revive(key, move);
                wait_for_vsync();
                pixel_buffer_start = *(pixel_ctrl_ptr + 1);
                break;

        }
        
    }

    return 0;
}



void draw_hero(int x, int y) {
    for (int i = 0; i < CHARACTER_SIZE; i++) {
        for (int j = 0; j < CHARACTER_SIZE; j++) {
            if (hero[i][j] != 0x0000) {
                plot_pixel(x + j, y + i, hero[i][j]);
            }
        }
    }
}

void clear_hero(int x, int y) {
    for (int i = 0; i < CHARACTER_SIZE; i++) {
        for (int j = 0; j < CHARACTER_SIZE; j++) {
            plot_pixel(x + j, y + i, background[y + i][x + j]);  // Restore background
        }
    }
}

void draw_enemy(int x, int y) {
    for (int i = 0; i < CHARACTER_SIZE; i++) {
        for (int j = 0; j < CHARACTER_SIZE; j++) {
            if (enemy[i][j] != 0x0000) {
                plot_pixel(x + j, y + i, enemy[i][j]);
            }
        }
    }
}

void clear_enemy(int x, int y) {
    for (int i = 0; i < CHARACTER_SIZE; i++) {
        for (int j = 0; j < CHARACTER_SIZE; j++) {
            plot_pixel(x + j, y + i, background[y + i][x + j]);  // Restore background
        }
    }
}

void draw_background() {
    for (int y = 0; y < SCREEN_HEIGHT; y++) {
        for (int x = 0; x < SCREEN_WIDTH; x++) {
            plot_pixel(x, y, background[y][x]);
        }
    }
}

void draw_game_over_screen(void) {
    // Clear the screen.
    clear_screen();

    // Prepare the game over message.
    char gameOverMsg[] = "GAME OVER";
    int textWidth = strlen(gameOverMsg) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 100;  // Vertical center-ish.
    draw_string(xPos, yPos, gameOverMsg, RED);

    // Optionally, draw instructions for reviving.
    char instructions[] = "Press all keys to revive";
    int instWidth = strlen(instructions) * CHAR_WIDTH;
    int xInst = (320 - instWidth) / 2;
    int yInst = yPos + 20;
    draw_string(xInst, yInst, instructions, GREEN);
}

void draw_game_start_screen(void)  {
    // Clear the screen.
    clear_screen();

    // Prepare the game over message.
    char gameOverMsg[] = "Pac-Man Upside Down";
    int textWidth = strlen(gameOverMsg) * CHAR_WIDTH;
    int xPos = (320 - textWidth) / 2;
    int yPos = 100;  // Vertical center-ish.
    draw_string(xPos, yPos, gameOverMsg, RED);

    // Optionally, draw instructions for reviving.
    char instructions[] = "Warning: Include Violent Content (18+)";
    int instWidth = strlen(instructions) * CHAR_WIDTH;
    int xInst = (320 - instWidth) / 2;
    int yInst = yPos + 20;
    draw_string(xInst, yInst, instructions, GREEN);
}

void check_mutation(void) {
    int mutation_chance = 1000;
    int mutation_range = 2 + difficulty;

    for (int i = 0; i < 3; i++) {
        int mutation_index = rand() % (mutation_chance+ 1);
        if (mutation_index <= mutation_range) {
            enemyY[i] -= enemyHeight;
            draw_box(enemyX[i], enemyY[i], enemyLenght, enemyHeight, BLACK);
        }
    }


}

void mutation(int original_height_index) {
    if(original_height_index == 0) {
        original_height_index += 1;
    } else if(original_height_index == 3) {
        original_height_index -= 1;
    } else {
        // Randomly choose between +1 and -1 (50/50 chance)
        int change = (rand() % 2 == 0) ? 1 : -1;
        original_height_index += change;
    }
}

void check_start(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    // Check left key for revival
    if(move){
        if(((key_val & 0x8) != 0 && (key_val & 0x4) != 0 && (key_val & 0x2) != 0) || key==0x29){
            clear_screen();
            score = 0;
            for (int i = 0; i < 6; i++) {
                draw_box(enemyX[i], enemyY[i], enemyLenght, enemyHeight, BLACK);
            }
            enemyX[0] = 320;
            enemyX[1] = 400;
            enemyX[2] = 480;
            // Dungeon Enemies
            enemyX[3] = 0;
            enemyX[4] = -80;
            enemyX[5] = -160;
            gameOn = 1;
        }
    }
}

void check_revive(char key, int move) {
    int enemyAltitiude = groundHeight - enemyHeight -1;
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys

    // Check left key for revival
    if(move){
        if (((key_val & 0x8) != 0 && (key_val & 0x4) != 0 && (key_val & 0x2) != 0 && (key_val & 0x1) != 0 ) || (key == 0x29)) {
            clear_screen();
            score = 0;
            for (int i = 0; i < 3; i++) {
                draw_box(enemyX[i], enemyY[i], enemyLenght, enemyHeight, BLACK);
            }
            for (int i = 3; i < 6; i++) {
                draw_slime(enemyX[i], enemyY[i], enemyLenght, enemyHeight, BLACK);
            }
            enemyX[0] = 320;
            enemyX[1] = 400;
            enemyX[2] = 480;
            // Dungeon Enemies
            enemyX[3] = 0;
            enemyX[4] = -80;
            enemyX[5] = -160;
            clear_both_buffers();
            gameOn = 0;
        }        
    }

}

void check_collision(int enemyX[], int enemyY[], int heroX, int heroY) {
    for(int i = 0; i < 6; i++) {
            int e_x1 = enemyX[i] + 3;
            int e_x2 = enemyX[i] + enemyLenght - 3;
            int e_x3 = enemyX[i] + 0.5 * enemyLenght; 
            int x1 = heroX - e_x1;
            int x2 = heroX - e_x2;
            int x3 = heroX - e_x3;
            int y = heroY - enemyY[i];
            int y_half = heroY - enemyY[i] - enemyHeight * 0.5;
            int y_full = heroY - enemyY[i] - enemyHeight;

            double l1 = x1 * x1 + y * y;
            double l2 = x2 * x2 + y * y;
            double l3 = x3 * x3 + y_half * y_half;
            double l4 = x1 * x1 + y_full * y_full;
            double l5 = x2 * x2 + y_full * y_full;

            if (l1 < heroRadius * heroRadius || l2 < heroRadius * heroRadius || l3 < heroRadius * heroRadius
                || l4 < heroRadius * heroRadius || l5 < heroRadius * heroRadius) {

                // Death Animation    
                animate_ball_crack(heroX, heroY, heroRadius);
                leaderboardUpdated = 0;
                gameOn = 2;
        
        }


    }
     
}


void draw_char(int x, int y, char c, short int color) {
    if (c < 32 || c > 127)
        return;  // Only support ASCII 32 to 127.
    int row, col;
    int index = c - 32;
    for (row = 0; row < CHAR_HEIGHT; row++) {
        for (col = CHAR_WIDTH; col > 0; col--) {
            // Check if the bit is set in the bitmap for this row.
            if (font[index][row] & (1 << (7 - col))) {
                plot_pixel(x + col, y + row, color);
            }
        }
    }
}

void draw_string(int x, int y, const char* str, short int color) {
    while (*str) {
        draw_char(x, y, *str, color);
        x += CHAR_WIDTH; // Move x for the next character.
        str++;
    }
}



int update_enemy_positions(int amplitude) {
    int i;
    for (i = 0; i < 3; i++) {

        // Up Button
        if (amplitude > THRESHOLD) {
            if (enemyY[i] >= groundHeight - enemyHeight) {
                enemy_up_velocity = -15;  // Set initial upward velocity for jump
            }
            amplitude = 0;
        }


        // Apply gravity every frame
        enemy_up_velocity += enemy_gravity;
        enemyY[i] += enemy_up_velocity;

        // Prevent the hero from falling below the ground
        if (enemyY[i] > groundHeight  - enemyHeight) {
            enemyY[i] = groundHeight - enemyHeight;
            enemy_up_velocity = 0;
        }
        
        // Int division.
        difficulty = 1 + score / 3;

        // Move enemy left by 1 pixel.
        enemyX[i] -= (1 + difficulty);

        // If enemy goes off the left edge of the screen, reset its position.
        if (enemyX[i] < 0) {
            enemyX[i] = 320 + rand() % (450 - 320 + 1);

            clear_score();

            enemyY[i] = groundHeight - enemyHeight;

            // Increment the score when the enemy object destroyed.
            score += 1;
        }
    }

    for (i = 3; i < 6; i++) {
        
        // Int division.
        difficulty = 1 + score / 3;

        // Cap max difficulty at 5;
        if (difficulty >= 5) difficulty = 5;

        // Move enemy left by 1 pixel.
        enemyX[i] += (1 + difficulty);

        // If enemy goes off the left edge of the screen, reset its position.
        if (enemyX[i] > 320) {
            enemyX[i] =  - rand() % (0 - 130 + 1);

            clear_score();

            enemyY[i] = groundHeight + 1;

        }
    }
    return amplitude;
}

// Plots a single pixel at (x, y) with the specified color.
void plot_pixel(int x, int y, short int line_color)
{
    volatile short int * pixel_address;
    pixel_address = (volatile short int *)(pixel_buffer_start + (y << 10) + (x << 1));
    *pixel_address = line_color;
}

// Clears the entire screen by setting every pixel to black.
void clear_screen()
{
    int x, y;
    for (y = 0; y < 240; y++)
    {
        for (x = 0; x < 320; x++)
        {
            plot_pixel(x, y, 0); // Black color.
        }
    }
}

void clear_score() {
    int x, y;
    for (y = 0; y < 20; y++)
    {
        for (x = 300; x < 320; x++)
        {
            plot_pixel(x, y, 0); // Black color.
        }
    }
}

void clear_score_left() {
    int x, y;
    for (y = 0; y < 20; y++)
    {
        for (x = 125; x < 150; x++)
        {
            plot_pixel(x, y, 0); // Black color.
        }
    }
}

// Draws a horizontal line across the entire screen at the given row.
void draw_horizontal_line(int row, short int color)
{
    int x;
    for (x = 0; x < 320; x++)
    {
        plot_pixel(x, row, color);
    }
}

// Draw a color box (enemy)
void draw_box(int locationX, int locationY, int length, int height, short int color) {
    draw_enemy(locationX, locationY);
}

void draw_slime(int x, int y, int length, int height, short int color) {
    for (int i = 0; i < CHARACTER_SIZE; i++) {
        for (int j = 0; j < CHARACTER_SIZE; j++) {
            if (slime[i][j] != 0x0000) {
                plot_pixel(x + j, y + i, slime[i][j]);
            }
        }
    }
}

// Wait for a vertical sync by writing 1 to the Buffer register and then waiting until the S bit (bit 0)
// in the Status register (at address 0xFF20302C) becomes 0.
void wait_for_vsync()
{
    volatile int * pixel_ctrl_ptr = (int *)0xFF203020;
    volatile int * status = (int *)0xFF20302C;
    
    // Write 1 to the Buffer register to initiate a buffer swap / vsync synchronization.
    *pixel_ctrl_ptr = 1;
    
    // Wait until the S bit (bit 0) of the status register is cleared.
    while ((*status & 0x01) != 0)
    {
        ; // do nothing, just wait
    }
}


// Draws a circle with center (x_center, y_center) and radius using the specified color.
void draw_circle(int x_center, int y_center, int radius, short int color)
{

    draw_hero(x_center, y_center - radius);
}

void update_hero_position(char key, int move) {
    volatile int * key_ptr = (volatile int *)0xFF200050;  // Address for DE-SOC keys (verify with your board documentation)
    int key_val = *key_ptr & 0xF;  // Use lower 4 bits for the 4 keys
    if(move){
        // Down Button
        if ((key_val & 0x1) != 0 || (key == 0x1B)) {
            // If hero is on the top side
            if (side == 0) {
                // Only if hero is current on the ground
                if (heroY >= groundHeight - heroRadius - 1) {
                    side = 1;
                    heroY = groundHeight + heroRadius + 1;
                }
            } else {
                // If hero is on the bottom side and on the ground
                if (heroY <= groundHeight + heroRadius + 1 ) {
                    hero_velocity = 20;
                }
            }

        }

        // Up Button
        if ((key_val & 0x2) != 0 || (key == 0x1D)) {

            if (side == 0) {
                if (heroY >= groundHeight - heroRadius - 1) {
                    hero_velocity = -20;  // Set initial upward velocity for jump
                }
            } else {
                // If hero is on the bottom side and wants to switch up
                if (heroY <= groundHeight + heroRadius + 1 ) {
                    side = 0;
                    // switch hero position
                    heroY = groundHeight - heroRadius;
                    
                }
            }


        }

        // Right Button
        if ((key_val & 0x4) != 0|| (key == 0x23)) {

            heroX += 5;
            if (heroX > 320 - heroRadius) {
                heroX = 320 - heroRadius - safeDistance;
            }


        }

        // Left Button
        if ((key_val & 0x8) != 0 || (key == 0x1C)) {
            heroX -= 5;
            if (heroX < 0) {
                heroX = heroRadius + safeDistance;
            }
        }
    }



    // If on side 0, apply gravity; else apply anti-gravity
    if (side == 0) {
        // Apply gravity every frame
        hero_velocity += gravity;
        heroY += hero_velocity;

        // Prevent the hero from falling below the ground
        if (heroY > groundHeight - heroRadius - 1) {
            heroY = groundHeight - heroRadius - 1;
            hero_velocity = 0;
        }        
    } else {
        // Apply anti-gravity every frame
        hero_velocity -= gravity;
        heroY += hero_velocity;

        // Prevent the hero from flying above4 the ground
        if (heroY <= groundHeight + heroRadius + 1) {
            heroY = groundHeight + heroRadius + 1;
            hero_velocity = 0;
        } 
    }

}

// Draws a line using Bresenham's algorithm.
void draw_line(int x0, int y0, int x1, int y1, short int color) {
    int dx = abs(x1 - x0), sx = (x0 < x1) ? 1 : -1;
    int dy = -abs(y1 - y0), sy = (y0 < y1) ? 1 : -1;
    int err = dx + dy, e2;
    while (1) {
        plot_pixel(x0, y0, color);
        if (x0 == x1 && y0 == y1)
            break;
        e2 = 2 * err;
        if (e2 >= dy) { err += dy; x0 += sx; }
        if (e2 <= dx) { err += dx; y0 += sy; }
    }
}

// Simple delay function: waits for a given number of vertical sync frames.
void delay(int frames) {
    for (int i = 0; i < frames; i++) {
        wait_for_vsync();
    }
}

// Animates the hero ball "cracking" by drawing a series of frames.
// You can call this function with the hero's center coordinates and radius.
void animate_ball_crack(int centerX, int centerY, int radius) {
    // Frame 1: Draw intact ball with a single crack.
    //clear_screen();  // Clear the back buffer.
    draw_circle(centerX, centerY, radius, PINK);
    draw_line(centerX, centerY, centerX - radius, centerY - radius, RED);
    delay(10);

    // Frame 2: Add a second crack.
    //clear_screen();
    draw_circle(centerX, centerY, radius, PINK);
    draw_line(centerX, centerY, centerX - radius, centerY - radius, RED);
    draw_line(centerX, centerY, centerX + radius, centerY + radius, RED);
    delay(10);

    // Frame 3: Draw additional cracks.
    //clear_screen();
    draw_circle(centerX, centerY, radius, PINK);
    draw_line(centerX, centerY, centerX - radius, centerY - radius, RED);
    draw_line(centerX, centerY, centerX + radius, centerY + radius, RED);
    draw_line(centerX, centerY, centerX - radius, centerY + radius, RED);
    draw_line(centerX, centerY, centerX + radius, centerY - radius, RED);
    delay(10);
    
    animate_blood_splash(centerX, centerY);
    animate_blood_splash(centerX, centerY);
    delay(10);

    // Instead of a final clear, simply swap buffers once more.
    wait_for_vsync();
    volatile int *pixel_ctrl_ptr = (int *)0xFF203020;
    pixel_buffer_start = *(pixel_ctrl_ptr + 1);

    animate_blood_drops(centerX, centerY);  

}

// Animates a blood splash by drawing several red streaks from (startX, startY).
void animate_blood_splash(int startX, int startY) {
    int splashCount = 20;  // Number of blood streaks.
    for (int i = 0; i < splashCount; i++) {
        // Random horizontal offset: -40 to 40.
        int offsetX = rand() % 81 - 40;
        // Random vertical drop: 20 to 60 pixels down.
        int offsetY = rand() % 41 + 20;
        int endX = startX + offsetX;
        int endY = startY + offsetY;
        draw_line(startX, startY, endX, endY, RED);
    }
}

void animate_blood_drops(int centerX, int centerY) {
    BloodDrop drops[NUM_DROPS];
    int i, frame;

    // Initialize each drop at the center with random velocity.
    for (i = 0; i < NUM_DROPS; i++) {
        drops[i].x = centerX;
        drops[i].y = centerY;
        drops[i].vx = (rand() % 11 - 5);  // Range: -5 to 5
        drops[i].vy = (rand() % 11 - 7);  // Range: -7 to 3 (bias upward)
    }

    // Animate for a set number of frames.
    for (frame = 0; frame < 100; frame++) {
        // Clear the current back buffer before drawing this frame.
        clear_screen();

        // Update and draw each drop.
        for (i = 0; i < NUM_DROPS; i++) {
            // Apply a simple gravity effect.
            drops[i].vy += 0.2;
            drops[i].x += drops[i].vx;
            drops[i].y += drops[i].vy;

            // Draw the drop if within screen bounds.
            int px = (int) drops[i].x;
            int py = (int) drops[i].y;
            if (px >= 0 && px < 320 && py >= 0 && py < 240) {
                plot_pixel(px, py, RED);
            }
        }

        // Optionally, you might want to draw additional blood effects here.
        // For example, drawing a horizontal red line that moves:
        //draw_horizontal_line(240 - frame, RED);

        // Swap buffers to display the current frame.
        wait_for_vsync();
        // After vsync, update the global pixel_buffer_start pointer.
        volatile int *pixel_ctrl_ptr = (int *)0xFF203020;
        pixel_buffer_start = *(pixel_ctrl_ptr + 1);
    }
}

void animate_revive(int centerX, int centerY, int finalRadius) {
    int numFrames = 20;  // Total frames in the animation
    // Precomputed unit-like vectors for 8 directions:
    int dx[8] = { 1,  1,  0, -1, -1, -1,  0,  1};
    int dy[8] = { 0, -1, -1, -1,  0,  1,  1,  1};
    int offset = 5; // Additional length for the glow effect

    animate_reverse_blood_spring(centerX, centerY);

    for (int frame = 0; frame <= numFrames; frame++) {
        // Compute the current radius (growing from 0 to finalRadius)
        int currentRadius = (frame * finalRadius) / numFrames;

        // Clear the back buffer for the new frame.
        clear_screen();
        
        // Draw the expanding hero ball.
        draw_circle(centerX, centerY, currentRadius, PINK);
        
        
        // Swap buffers to display this frame.
        wait_for_vsync();
        // Update the global pixel_buffer_start to the new back buffer pointer.
        volatile int *pixel_ctrl_ptr = (int *)0xFF203020;
        pixel_buffer_start = *(pixel_ctrl_ptr + 1);
        
        // Optional: Delay to control animation speed.
        delay(2);
    }
}
