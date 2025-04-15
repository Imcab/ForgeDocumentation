# Math - Forge

## Introduction

`Math` is a feature designed to enhance robotics control systems, offering smooth double supplier outputs, boolean triggers, and advanced angle operations like coterminal and atan3. It provides flexible PID wrapper classes, feedforward options, and motion control with trapezoidal profiles and cascade control. Specialized constants for PID, PIDF, SimpleFeedforward, CompleteFeedForward, and MotionModel allow for precise control.

## ProfileGains

Used for storing different types of profiles, each one with own Sendables for easy showing on dashboard.

### Creating a gains object

``` java

    PIDGains turnPIDGains = new PIDGains(8.0, 0,0);
    PIDGains drivePIDGains = new PIDGains(0.05, 0, 0);
    SimpleFeedForwardGains driveFFGains = new SimpleFeedForwardGains(0, 0.0789, 0); 

```

!!! info
    Check for all the supported types!

### Publishing

If you want to show the stored gains in the dashboard you can just do:

``` java

    PIDGains turnPIDGains = new PIDGains(8.0, 0,0);
    
    SmartDashboard.putData("MyGains", turnPIDGains); //Send it to dashboard

```

!!! warning
    Don't publish the gains in a periodic() or methods that updates constantly, we recommend using the constructor!

In addition, this class support creating gains from an array of doubles or transforming the gains into an array object.

## Control

The Control class provides different types of controllers (e.g., PID control, motion model control, etc.) to manage control outputs. It also supports chaining operations on control results, allowing more flexible control designs.

### ControlResult

All calculate() methods of Controls return an `ControlResult`, that represents an output of a control system. You can chain various operations such as addition, subtraction, clamping, etc., on the result.

#### Swerve module Example

``` java

    public void periodic(){
        
        if (angleSetpoint != null) {
            this.turnVoltage = turnPID.calculate(moduleAngle.getRadians()).getOutput();

            if (speedSetpoint != null) {
                //Chains a method
                this.driveVoltage = drivePID.calculate(driveVelocity).plus(()-> ffVolts).getOutput();
            }else{
                drivePID.reset();
            }
        }else{
            turnPID.reset();
        }
    }

```

#### Generic example

``` java

    ControlResult result = pid.calculate(moduleAngle.getRadians(), 90)
    .clamp(-10, 10) //Clamp the result to range [-10, 10]
    .times(2) //Multiply the result by 2
    .negate(); //Inverts the output

    double finalOutput = result.getOutput(); //Final output after all operations


```

### Usage of Controllers

### Creation

Creating a control is preatty easy, just create it as you created it from wpilib but instead of pasing each
gain parameter kP, kI, kD, you can just pass the entire gains:

Creating a simple PID:

``` java
    //The corresponding gains
    PIDGains turnPIDGains = new PIDGains(5.0, 0,0);    

    //Creating a PID with the given gains
    PIDControl turnPID = new PIDControl(turnPIDGains);
```

Creating a motionModel (Trapezoidal control) with new gains:

``` java
    //Individual parameters
    private static final double ANGLE_KP = 5.0;
    private static final double ANGLE_KD = 0.4;
    private static final double ANGLE_MAX_VELOCITY = 8.0;
    private static final double ANGLE_MAX_ACCELERATION = 20.0;   

    //Creating a MotionModel with the new gains
    MotionModelControl angleController = new MotionModelControl(
            new MotionModelGains(
                ANGLE_KP,
                0,
                ANGLE_KD,
                ANGLE_MAX_VELOCITY,
                ANGLE_MAX_ACCELERATION));

```

Cascade control is a custom Forge controller, supporting two PIDLoops for getting a result

``` java
    private final PIDControl PositionControl = new PIDControl(outerGains);
    private final PIDControl VelocityControl = new PIDControl(innerGains);

    private final CascadeControl controller = new CascadeControl(PositionControl, VelocityControl);
```

#### Usage

For using controller such as pid, trapezoidal control (motionmodel) or others, is just like wpilib classes:

Using pid controller

``` java
    turnPID.calculate(currentRotation.getRadians(), moduleAngle.getRadians()).getOutput();
```

For using Feedforward, it is an static method as it is not as complex as other control models, returns a `ControlResult` based on a given velocity and acceleration:

``` java
    //The feedforward gains to calculate the output
    SimpleFeedForwardGains driveFFGains = new SimpleFeedForwardGains(0, 0.0789, 0); 

    //Calculate based on a given velocity and acceleration
    ffVolts = FeedforwardControl.calculate(driveFFGains, velocity, acceleration).getOutput();
```

Using a motion model:

``` java
    double omega =
                angleController.calculate(
                        new PositionState(angleSupplier.get().getRadians()),
                        drive.getRotation().getRadians()
                ).getOutput();
```

For MotionModel to work you need to pass a `Trapezoidal.State` as setpoint which includes position and velocity values,
if you only want to use it for position you can just create a `PositionState` and pass the desired position as the example above

All methods of common controllers have the hability to set Tolerance, continous input, etc. Except for CascadeControl

``` java
    //Enabling continousInput
    angleController.continuousInput(-Math.PI, Math.PI);

    //Setting velocity and position tolerance for a MotionModel profile
    angleController.setTolerance(0.01, 0);

```

## SmoothDoubleSupplier

The `SmoothDoubleSupplier` class provides a way to filter and smooth joystick inputs in FRC (FIRST Robotics Competition) Java programming. It is designed to improve robot control by reducing joystick sensitivity at low values while still allowing full range motion. This is particularly useful for swerve drive, tank drive, elevator control, and other mechanisms that require precise input adjustments.

Why Use SmoothDoubleSupplier?

- Enhances joystick control by modifying input response.

- Prevents sudden jerks in movement.

- Improves precision in mechanisms like arms, elevators, and intakes.

- Customizable filtering using parameters (kE, kI, kD).

For properly tuning smooth constants, consider:

- Exponential Filtering (kE): Applies a power function to smooth inputs
- Offset (kI): Constant offset
- Additional Power Component (kD): Includes another power term for finer adjustments

!!! info
    You **don't** always need to use all the constants, with kE you can get a pretty result

If kE > 1, small values are further reduced, enhancing low-speed precision.

If kE < 1, small values are amplified, making the joystick more responsive.

### Smoothing an input

To smooth an input just call the *apply()* method to the DoubleSupplier you want:

``` java
    //Apply smoothing to the Y-axis input
    SmoothDoubleSupplier.apply(()-> -driver.getLeftY(), 1.5);
```

## Operator

The `Operator` class provides a collection of mathematical operations as functional interfaces. It is designed to simplify common mathematical computations in FRC, particularly for joystick input processing, robot movement calculations, and general mathematical utilities.

- Binary operations (Operation) like addition, subtraction, and power functions.

- Unary operations (UnaryOperation) like negation, absolute value, and squaring.

- Custom mathematical utilities for handling angles and deadbanding joystick input.

### Available operations

#### Binary Operations (Operation)

| Operation   | Description                          |
| ----------- | ------------------------------------ |
| `ADD`       | Adds two values |
| `SUBSTRACT` | Subtracts second value from first |
| `MULTIPLY`    | Multiplies two values |
| `DIVIDE`    | Divides first value by second |
| `MODULO`    | Remainder of division |
| `POWER`    | Raises first value to the power of second |
| `DISTANCE`    | Absolute difference between two values |
| `MEAN`    | Average of two values |
| `DEADBAND`    | Zeroes values below a threshold |

#### Unary Operations (UnaryOperation)

| Operation   | Description                          |
| ----------- | ------------------------------------ |
| `SQUARE_ROOT`       | Computes square root |
| `SQUARE` | Squares a number |
| `NEGATE`    | Negates the value (Inverts) |
| `ABSOLUTE`    | Converts to absolute value |
| `INVERSE`    | Computes reciprocal |
| `INCREMENT`    | Increases value by 1 |
| `DECREMENT`    | Decreases value by 1 |
| `ROUND`    | Rounds to nearest whole number |

#### Angle-Specific Operations

| Operation   | Description                          |
| ----------- | ------------------------------------ |
| `COTERMINALRADIANS`       | Converts an angle to its coterminal equivalent in radians |
| `COTERMINALDEGREES` | Converts an angle to its coterminal equivalent in degrees |
| `MINANGLEDEGREES`    | Finds the smallest angle difference between two degrees values|
| `MINANGLERADIANS`    | Finds the smallest angle difference between two radian values |

### Examples

Examples of some of the many Operator methods available

#### Deadbanding Joystick Input

Removes small joystick noise by zeroing inputs below 0.05.

``` java

    double joystickInput = -driver.getLeftY();  
    double deadbandedInput = Operator.DEADBAND.apply(joystickInput, 0.05); 


```

#### Joystick drive Command

Use Operator to square values:

``` java

    public static Command joystickDrive(
        Holonomic drive,
        DoubleSupplier xSupplier,
        DoubleSupplier ySupplier,
        DoubleSupplier omegaSupplier){
    return Commands.run(
        () -> {
            // Apply deadband
            Translation2d linearVelocity = getLinearVelocityFromJoysticks(xSupplier.getAsDouble(), ySupplier.getAsDouble());

            double omega = MathUtil.applyDeadband(omegaSupplier.getAsDouble(), DEADBAND);

            // Square values
            omega = Math.copySign(Operator.SQUARE.apply(omega), omega);

            ChassisSpeeds speeds =
                new ChassisSpeeds(
                    linearVelocity.getX() * drive.getMaxLinearSpeedMetersPerSec(),
                    linearVelocity.getY() * drive.getMaxLinearSpeedMetersPerSec(),
                    omega * drive.getMaxAngularSpeedRadPerSec());

            drive.runVelocity(
              ChassisSpeeds.fromFieldRelativeSpeeds(
                  speeds,
                  AllianceUtil.isRed()
                      ? drive.getRotation().plus(new Rotation2d(Math.PI))
                      : drive.getRotation()));
        },
        drive);
    }

```

#### Normalizing Angles

Ensures an angle is within 0 to 360 degrees:

``` java

    double angle = 450;  
    double normalizedAngle = Operator.COTERMINALDEGREES.apply(angle);  

```

### Using Custom Operations

Custom operations can be applied to joystick inputs, sensor data, or motor outputs.

``` java

        private final Joystick joystick = new Joystick(0);

        //Custom operation: raises joystick input to the power of 3
        private final UnaryOperation cubicScaling = (a) -> a * a * a;

        public void teleopPeriodic() {
            double joystickValue = joystick.getY();
            double processedValue = cubicScaling.apply(joystickValue);
        }
    
```

## HPPMathLib

The `HPPMathLib` class provides utility methods for angle calculations in both degrees and radians. It includes functions for computing coterminal angles, finding the minimum angle difference, and a specialized `atan3` function.

### Coterminal

Returns the coterminal angle of angle in degrees, ensuring it is within [0, 360) or [0, 2π).

=== "Degrees"

    ``` java

        double normalizedAngle = HPPMathLib.coterminalDegrees(450);  
    ```

=== "Radians"

    ``` java

        double normalizedAngle = HPPMathLib.coterminalRadians(5.0);  
    ```

### MinAngle

Finds the minimum angular difference between ang_from and ang_to

=== "Degrees"

    ``` java
     
        double minAngle = HPPMathLib.MinAngleDegrees(a, b); 
    ```

=== "Radians"

    ``` java
        
        double minAngle = HPPMathLib.MinAngleRadians(a, b); 
    ```

### Atan 3

Computes the arctangent of y/x while considering a threshold to return a default value if the input is too small.

=== "Degrees"

    ``` java
        
        double result = HPPMathLib.atan3Degrees(a, b, 0.1, 0);  
    ```

=== "Radians"

    ``` java
        
        double result = HPPMathLib.atan3Radians(a, b, 0.1, 0);   
    ```

## BooleanTrigger

The `BooleanTrigger` class is used to track and respond to changes in boolean states over time. It is useful in scenarios where you need to detect transitions, such as detecting when a boolean condition becomes true, false, or changes state.

``` java

    BooleanTrigger trigger = new BooleanTrigger(() -> someBooleanCondition);

    if (trigger.onTrue()) {
        //Do something when the condition just became true
    }

    if (trigger.onFalse()) {
        //Do something when the condition just became false
    }

    if (trigger.onChange()) {
        //Do something when the condition changes
    }

    BooleanTrigger anotherTrigger = new BooleanTrigger(() -> anotherCondition);
    BooleanTrigger combinedTrigger = trigger.and(anotherTrigger);

    if (combinedTrigger.onTrue()) {
        //Do something when both conditions are true
    }

```
