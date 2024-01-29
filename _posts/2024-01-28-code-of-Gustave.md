---
layout: post
title: Code of Gustave
subtitle: The source code and the technical details
author: Van David LE
---

# Elemental pieces of code used:

## Moving

### Moving forward and backward

Function to move Gustave forward. Includes active correction of direction based on gyroscope values. Returns the distance travelled by Gustave. \
Author : Alban
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
Function to move Gustave backwards. Works pretty much the same way as move_forward, with the same active correction. Also returns the distance travelled. \
Author : Alban
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
Function to update the x and y variables based on the distance travelled and the direction. \
Author : Alban
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
Helper function to run both motors at given speeds forever. \
Author : Guillaume
```c
void run_forever(uint8_t sn1, uint8_t sn2, int speed_sn1, int speed_sn2){
    
    set_tacho_speed_sp(sn1, speed_sn1);
    set_tacho_speed_sp(sn2, speed_sn2);
    set_tacho_command_inx(sn1, TACHO_RUN_FOREVER);
    set_tacho_command_inx(sn2, TACHO_RUN_FOREVER);
}
```
Function to move the robot to the flag. It will make him move sideways and forward. \
Author : Alban
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

Function to rotate the robot at a given angle, speed and direction. If direction is true, robot will turn clockwise. 
Returns the angle value at which the robot turned (negative angle if counterclockwise rotation). \
Author : Alban

```c
nt rotate(uint8_t sn1, uint8_t sn2, uint8_t snGyro, int angle, int max_speed, bool right){
    int minus = 1;
    if (!right){
        minus = -1;
    }
    float gyro_value = get_gyro_value(snGyro);
    run_forever(sn1,sn2, minus * max_speed * 0.5, -minus * max_speed * 0.5)
    bool called = false;
    while((get_gyro_value(snGyro) < gyro_value + angle && right) || (get_gyro_value(snGyro) > gyro_value - angle && !right)){
        if (((get_gyro_value(snGyro) > gyro_value + angle - 40 && right)||(get_gyro_value(snGyro) < gyro_value - angle + 40 && !right)) && called == false){
            called = true;
            run_forever(sn1,sn2, minus * max_speed * 0.1, -minus * max_speed * 0.1)
        }
        sleep(0.01);
    }
    set_tacho_command_inx(sn1, TACHO_STOP);
    set_tacho_command_inx(sn2, TACHO_STOP);
    return minus * angle;
}

```
Function to retrieve the orientation of the robot. 0 is the robot is straight, 1 if turned to the right, 2 if turned backwards, 3 if turned to the left. \
Author: Alban
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

## Claw

```c
/* Helper function to use the robot's arm/claw.Negative degress pull the arm towards the robot 
Author : Guillaume*/
void claw(uint8_t sn_arm, int degrees, int speed){
    set_tacho_speed_sp(sn_arm, max_speed);
    set_tacho_stop_action_inx(sn_arm , TACHO_BRAKE);
    set_tacho_position_sp(sn_arm, degrees);
    set_tacho_command_inx(sn_arm, TACHO_RUN_TO_REL_POS);
    sleep(1);
    set_tacho_command_inx(sn_arm, TACHO_STOP);
}
```

# Main algorithm that runs during the round

```c
int main(void) {
    uint8_t sn1, sn2, sn_arm, snUS, snGyro;
    int max_speed1, max_speed2, max_speed_arm;
    // Initialize libraries
    if (ev3_init() < 1) {
        return 1;
    }
    printf("EV3 init succeeded\n");
    ev3_tacho_init();
    
    // Trouver le premier moteur sur le port sp  cifique
    if (ev3_search_tacho_plugged_in(PORT_B, EXT_PORT_B, &sn1, 0) == 0) {
        printf("Wheel motor 1 not found on port B :((\n");
        return 1;
    }
    
    // Trouver le deuxi  me moteur sur le port sp  cifique
    if (ev3_search_tacho_plugged_in(PORT_C, EXT_PORT_C, &sn2, 0) == 0) {
        printf("Wheel motor 2 not found on port C :((\n");
        return 1;
    }
    
    if (ev3_search_tacho_plugged_in(PORT_A, EXT_PORT_A, &sn_arm, 0) == 0) {
        printf("Arm motor not found on port A :((\n");
        return 1;
    }
    
    printf("All motors found ! \n");
    
    // Retrieving motor's maximum speed
    get_tacho_max_speed(sn1, &max_speed1);
    get_tacho_max_speed(sn2, &max_speed2);
    get_tacho_max_speed(sn_arm, &max_speed_arm);
    int max_speed = max_speed2;
    if (max_speed1 < max_speed2){
        max_speed = max_speed1;
    }
    max_speed = max_speed / 2;
    
    // Initializing all needed sensors
    ev3_sensor_init();
    if (ev3_search_sensor(LEGO_EV3_US, &snUS, 0) == 0){
        printf("Ultrasound sensor not found :((\n");
        return 1;
    }
    if (ev3_search_sensor(LEGO_EV3_GYRO, &snGyro, 0) == 0){
        printf("Gyroscopic sensor not found :((\n");
        return 1;
    }
    printf("All sensors found ! \n");
    
    float gyrovalue = get_gyro_value(snGyro);
    int orientation = 0;
    double x = 0, y = 0;
    int angle = 0;
    int distance = 0;
    int mode = 0;
    
    // Opening Gustave's arm
    claw(sn_arm,150,max_speed_arm / 4);
    section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 300, 400, true);
    while(mode == 1){
        // Gustave is in the first half of the arena (our area)
        if (y < 420){
            // Gustave is in the right half of the arena
            if (x>0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 200, 300, false);
            }
            // Gustave is in the left half of the arena
            else if (x<0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 200, 300, true);
            }
        } 
        // Gustave is in the second half of the arena (opponent's area)
        else if (y > 830){
            // Gustave is in the right half of the arena
            if (x>0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 230, false);
            }
            // Gustave is in the left half of the arena
            else if (x<0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 230, true);
            }
        } else {
            // Gustave has encountered a close obstacle
            if (x>0)
            {
                distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, y-380, &mode);
                position(&x, &y, distance, orientation);
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 250, 400, false);
            } else if (x<0)
            {
                distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, y-380, &mode);
                position(&x, &y, distance, orientation);
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 250, 400, true);
            }
        }
    }
    if (x > 20){
        section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 210, false);
    } else if (x < -20){
        section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 600, 210, true);
    }
    
    // Gustave is now in front of the flag, he has to rotate and grab it
    angle = rotate(sn1, sn2, snGyro, 170, max_speed, true);
    gyrovalue += angle;
    get_orientation(&orientation, angle);
    claw(sn_arm,-130,max_speed_arm);
    angle = rotate(sn1, sn2, snGyro, 10, max_speed, true);
    gyrovalue += angle;
    get_orientation(&orientation, angle);
    // Gustave has grabbed the flag, he has to go back to his area
    section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 100, true);
    while(mode == 1){
        if (y < 420)
        {
            // Gustave is in the first half of arena
            if (x>0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 250, true);
            }
            else if (x<0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 250, false);
            }
        } else if (y > 830)
        {
            //robot second half of arena
            if (x>0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 250, true);
            } else if (x<0)
            {
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 250, false);
            }
        } 
        else 
        {
            // Gustave has encountered a close obstacle
            if (x>0)
            {
                distance = -move_backward(sn1, sn2, snUS, snGyro, gyrovalue, max_speed, -y+380, &mode);
                position(&x, &y, distance, orientation);
                section_path(sn1, sn2, snUS, snGyro, &gyrovalue, &orientation, &x, &y, max_speed, &mode, 350, 200, true);
            } else if (x<0)
            {
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
    claw(sn_arm, -190, max_speed_arm / 4);
    sleep(2);
    claw(sn_arm, 150, max_speed / 4);
}
```
