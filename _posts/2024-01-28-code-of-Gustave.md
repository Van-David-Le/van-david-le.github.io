---
layout: post
title: Code of Gustave
subtitle: The source code and the technical details
author: Van David LE
---

# Elemental pieces of code used:

## Moving

### Moving forward and backward

This function takes in arguments the references to the motors, the ultra sonic sensor and the gyroscope. It also takes the distance and the mode in which the robot is. The mode is set by the `section_path` function that will be explained later.
```c
int move_forward(uint8_t sn1, uint8_t sn2, uint8_t snUS, uint8_t snGyro, float gyroval, int max_speed, int distance, int* mode){
    // This function makes the robot go forward for "distance"
    set_tacho_speed_sp(sn1, max_speed);
    set_tacho_speed_sp(sn2, max_speed);
    set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
    set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
    float init_US_value = get_US_value(snUS);
    float prev_US_value = init_US_value;
    float value;
    while (( value = get_US_value(snUS)) > distance){ // Moving forward
        if ( value < prev_US_value - 200){
            set_tacho_command_inx(sn1, TACHO_STOP);
            set_tacho_command_inx(sn2, TACHO_STOP);
            *mode = 1;
            return init_US_value - prev_US_value;
        }
        if (get_gyro_value(snGyro) > gyroval) { // Auto check of the direction while moving forward
            set_tacho_speed_sp(sn1, max_speed * 0.95);
            set_tacho_speed_sp(sn2, max_speed);
            set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
            set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
        } else if (get_gyro_value(snGyro) < gyroval) {
            set_tacho_speed_sp(sn1, max_speed);
            set_tacho_speed_sp(sn2, max_speed * 0.95);
            set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
            set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
        }
        prev_US_value = value;
        sleep(0.01);
    }
    if (get_US_value(snUS) < prev_US_value - 200){ // Here, the robot found an obstacle in its path, it stops and sets the mode to 1 while returning the distance traveled
        set_tacho_command_inx(sn1, TACHO_STOP);
        set_tacho_command_inx(sn2, TACHO_STOP);
        *mode = 1;
        return init_US_value - prev_US_value;
    }
    set_tacho_command_inx(sn1, TACHO_STOP); // If the robot reached the asked distance without encountering obsatcle, the function returns and the mode is set to 0
    set_tacho_command_inx(sn2, TACHO_STOP);
    *mode = 0;
    return init_US_value - get_US_value(snUS);
}
```
```c
int move_backward(uint8_t sn1, uint8_t sn2, uint8_t snUS, uint8_t snGyro, float gyroval, int max_speed, int distance, int* mode){
    // Works like the `move_forward` function but in reverse.
    set_tacho_speed_sp(sn1, -max_speed);
    set_tacho_speed_sp(sn2, -max_speed);
    set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
    set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
    float init_US_value = get_US_value(snUS);
    float prev_US_value = init_US_value;
    float value;
    while (( value = get_US_value(snUS)) < init_US_value + distance){
        if ( value > prev_US_value + 200){
            set_tacho_command_inx(sn1, TACHO_STOP);
            set_tacho_command_inx(sn2, TACHO_STOP);
            *mode = 1;
            return prev_US_value - init_US_value;
        }
        if (get_gyro_value(snGyro) < gyroval) {
            set_tacho_speed_sp(sn1, -max_speed * 0.95);
            set_tacho_speed_sp(sn2, -max_speed);
            set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
            set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
        } else if (get_gyro_value(snGyro) > gyroval) {
            set_tacho_speed_sp(sn1, -max_speed);
            set_tacho_speed_sp(sn2, -max_speed * 0.95);
            set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
            set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
        }
        prev_US_value = value;
        sleep(0.01);
    }
    if (get_US_value(snUS) > prev_US_value + 200){
        set_tacho_command_inx(sn1, TACHO_STOP);
        set_tacho_command_inx(sn2, TACHO_STOP);
        *mode = 1;
        return prev_US_value - init_US_value;
    }
    set_tacho_command_inx(sn1, TACHO_STOP);
    set_tacho_command_inx(sn2, TACHO_STOP);
    *mode = 0;
    return get_US_value(snUS) - init_US_value;
}
```
```c
void position(double* x, double* y, float dist, int orientation){
    // Actualizes the x and y values according to the distance and the orientation in argument
    if (orientation == 0){
        *y += dist;
    }
    else if (orientation == 1){
        *x += dist;
    }
    else if (orientation == 2){
        *y -= dist;
    }
    else if (orientation == 3){
        *x -= dist;
    }
}
```

```c
void section_path(uint8_t sn1, uint8_t sn2, uint8_t snUS, uint8_t snGyro, float* gyrovalue, int* orientation, double* x, double* y, int max_speed, int* mode, int move_thresh1, int move_thresh2, bool right){
    // This function acts like the hub that centralizes the data acquired from the different sensors and outputs of the functions. It is called at the end of every moving action to actualize the variables
    double temp_x = *x, temp_y = *y;
    int temp_orientation = *orientation;
    float temp_gyrovalue = *gyrovalue;
    int temp_mode = *mode;
    int angle = rotate(sn1, sn2, snGyro, 90, max_speed, right);
    temp_gyrovalue +=angle;
    int distance = move_forward(sn1, sn2, snUS, snGyro, temp_gyrovalue, max_speed, move_thresh1, &temp_mode);
    get_orientation(&temp_orientation, angle);
    position(&temp_x, &temp_y, distance, temp_orientation);
    angle = rotate(sn1, sn2, snGyro, 90, max_speed, !right);
    temp_gyrovalue += angle;
    distance = move_forward(sn1, sn2, snUS, snGyro, temp_gyrovalue, max_speed, move_thresh2, &temp_mode);
    get_orientation(&temp_orientation, angle);
    position(&temp_x, &temp_y, distance, temp_orientation);
    *x = temp_x;
    *y = temp_y;
    *orientation = temp_orientation;
    *gyrovalue = temp_gyrovalue;
    *mode = temp_mode;
    printf("orientation: %.1f", temp_orientation);
}
```
### Rotations

```c
int rotate(uint8_t sn1, uint8_t sn2, uint8_t snGyro, int angle, int max_speed, bool right){
    // Rotates the robot for a given angle
    int minus = 1;
    if (!right){
        minus = -1;
    }
    float gyro_value = get_gyro_value(snGyro);
    set_tacho_speed_sp(sn1, minus * max_speed * 0.5);
    set_tacho_speed_sp(sn2, -minus * max_speed * 0.5);
    set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
    set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
    bool called = false;
    while((get_gyro_value(snGyro) < gyro_value + angle && right) || (get_gyro_value(snGyro) > gyro_value - angle && !right)){
        if (((get_gyro_value(snGyro) > gyro_value + angle - 40 && right)||(get_gyro_value(snGyro) < gyro_value - angle + 40 && !right)) && called == false){
            called = true;
            set_tacho_speed_sp(sn1, minus * max_speed * 0.075);
            set_tacho_speed_sp(sn2, -minus * max_speed * 0.075);
            set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
            set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
        }
        sleep(0.01);
    }
    set_tacho_command_inx(sn1, TACHO_STOP);
    set_tacho_command_inx(sn2, TACHO_STOP);
    return minus * angle;
}
```
```c
void get_orientation(int* orientation, int angle){
    // Modifies the `orientation` varible according to the angle given
    int new = angle / 90;
    *orientation += new;
    if (*orientation == -1){
        *orientation = 3;
    }
    else if (*orientation == 4){
        *orientation = 0;
    }
}
```
## Sensors

### Ultra sonic sensor

```c
float get_US_value(uint8_t snUS){
    // Returns the value of the ultra sonic sensor
    float val;
    get_sensor_value0(snUS, &val);
    return val;
}
```

### Gyroscope

```c
float get_gyro_value(uint8_t snGyro){
    // Returns the value of the gyroscope
    float val;
    get_sensor_value0(snGyro, &val);
    return val;
}
```

# Main algorithm that runs during the round

```c
int main(void) {
    // The main function that runs on the robot when in the arena, it calls the other functions depending on the information gathered about the robot's surroundings
    uint8_t sn1, sn2, sn_arm;
    int max_speed1, max_speed2, max_speed_arm;
    // Library initialization
    if (ev3_init() < 1) {
    return 1;
    }
    printf("EV3 init succeeded\n");
    ev3_tacho_init();
    
    // Checking for the first motor
    if (ev3_search_tacho_plugged_in(PORT_B, EXT_PORT_B, &sn1, 0) == 0) {
    printf("Moteur non trouv   sur le port B\n");
    return 1;
    }
    
    // Checking for the second motor
    if (ev3_search_tacho_plugged_in(PORT_C, EXT_PORT_C, &sn2, 0) == 0) {
    printf("Moteur non trouv   sur le port C\n");
    return 1;
    }
    
    if (ev3_search_tacho_plugged_in(PORT_A, EXT_PORT_A, &sn_arm, 0) == 0) {
    printf("Moteur non trouv   sur le port A\n");
    return 1;
    }
    
    printf("Moteurs trouv  s\n");
    // Getting motors' maximum speeds
    get_tacho_max_speed(sn1, &max_speed1);
    get_tacho_max_speed(sn2, &max_speed2);
    get_tacho_max_speed(sn_arm, &max_speed_arm);
    int max_speed = max_speed2;
    if (max_speed1 < max_speed2){
        max_speed = max_speed1;
    }
    max_speed = max_speed / 2;
    ev3_sensor_init();
    uint8_t snUS;
    if (ev3_search_sensor(LEGO_EV3_US, &snUS, 0)){
        printf("Capteur US trouv  \n");
    }
    else{
        printf("Capteur US non trouv  \n");
    }
    uint8_t snGyro;
    if (ev3_search_sensor(LEGO_EV3_GYRO, &snGyro, 0)){
        printf("Capteur Gyro trouv  \n");
    }
    else{
        printf("Capteur Gyro non trouv  \n");
    }
    float gyrovalue = get_gyro_value(snGyro);
    int orientation = 0;
    double x = 0, y = 0;
    double e_x = 0, e_y = 0;
    int angle = 0;
    int distance = 0;
    int mode = 0;
    
    section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 300, 400, true);
    while(mode == 1){
        if (y < 420){
            //robot first half of arena
            if (x>0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 200, 300, false);
            } 
            else if (x<0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 200, 300, true);
            }
            } else if (y > 830){
            //robot second half of arena, will try to grab the flag
            if (x>0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 180, false);
            } 
            else if (x<0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 180, true);
            }
            } else {
            //robot stuck
            if (x>0){
              distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, y-380, &mode);
              position(&x, &y, distance, orientation);
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 250, 400, false);
            } else if (x<0){
              distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, y-380, &mode);
              position(&x, &y, distance, orientation);
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 250, 400, true);
            }
        }
    }
    if (x > 20){
        section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 280, false);
    } else if (x < -20){
        section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 280, true);
    }
    angle = rotate(sn1, sn2, snGyro, 180, max_speed, true);
    gyrovalue += angle;
    get_orientation(&orientation, angle);
    //flag grab
    printf("grabbing flag...\n");
    set_tacho_speed_sp(sn_arm, max_speed / 4);
    set_tacho_stop_action_inx(sn_arm , TACHO_BRAKE);
    set_tacho_position_sp(sn_arm, -150);
    set_tacho_command_inx(sn_arm, TACHO_RUN_TO_REL_POS);
    sleep(3);
    set_tacho_command_inx(sn_arm, TACHO_STOP);
    //flag grab
    //choose a side according to the expected position of the enemy robot,
    section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 100, true);
    while(mode == 1){
        if (y < 420){
            //robot in first half of arena
            if (x>0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, true);
            } 
            else if (x<0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, false);
            }
        } else if (y > 830){
            //robot second half of arena
            if (x>0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, true);
            } else if (x<0){
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, false);
            }
        } else {
            //robot stuck
            if (x>0){
              distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, -y+380, &mode);
              position(&x, &y, distance, orientation);
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, true);
            } else if (x<0){
              distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, -y+380, &mode);
              position(&x, &y, distance, orientation);
              section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, false);
            }
        }
    }
    // check if robot is at the right place: y < 0
    rotate(sn1, sn2, snGyro, 90, max_speed, true);
    //release flag
    printf("releasing flag...\n");
    set_tacho_position_sp(sn_arm, 150);
    set_tacho_command_inx(sn_arm, TACHO_RUN_TO_REL_POS);
    sleep(3);
    set_tacho_command_inx(sn_arm, TACHO_STOP);
    //turn to let flag fall
    }
```
