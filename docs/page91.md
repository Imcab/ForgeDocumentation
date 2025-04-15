# REV - Forge

## Getting started

Forge allows users to program a way to much easier the RevBlinkin lev driver based on all the
[available patterns](https://docs.revrobotics.com/rev-crossover-products/blinkin/gs/patterns) Rev offers.

Forge also haves its own `SparkMax` wrapper class

## Creating a REVBlinkin

To create a REVBlinkin just pass the pwm port on the RoboRIO an thats all!

``` java
    //Creating a REVBlinkin on PWM port 0
    public final REVBlinkin blinkin = new REVBlinkin(0);

```

### Setting a pattern

For setting a patter to the REVBlinkin, use the `setPattern()` method that accept a [pattern](https://docs.revrobotics.com/rev-crossover-products/blinkin/gs/patterns) from REV LED PATTERN TABLES

``` java
    //Setting a pattern
    blinkin.setPattern(PatternType.FireLarge);
```

### Getting a pattern

Forge's REVBlinkin class supports the option to get the current pattern of the device as an `optional`.

Checking if the current pattern is Aqua:

``` java
    //Get leds current pattern 
    Optional<PatternType> pattern = blinkin.getPattern();

    //Compare
    if (pattern.isPresent() && pattern.get().equals(PatternType.Aqua)) {
      System.out.println("Hey! I am Aqua");
    }

```

If you just want to use the `getPattern()` method to check if the blinkin has an specified patter you can use the
`hasPattern()` method:

``` java
    //Compare
    if (blinkin.hasPattern(PatternType.Aqua)) {
      System.out.println("Hey! I am Aqua");
    }

```

## ForgeSparkMax

`ForgeSparkMax` is a wrapper instance of  rev´s `SparkMax`, it includes new methods, easy dashoard support, integrated relative encoder and easy configuration

### Configuring an ForgeSparkMax

For full configuration use the `flashConfiguration` method which accepts an current limit, invertState, idleMode and voltage compensation

``` java

    //Creation, ForgeSparkMax can assume you're using MotorType.kBrushless
    ForgeSparkMax driveSparkMax = new ForgeSparkMax(DriveConstants.frontLeft.DrivePort);

    //Full configuration
    driveSparkMax.flashConfiguration(
            isDriveMotorInverted, //The boolean to invert the motor
            IdleMode.kBrake, //Idle mode
            43, //Current limit
            true); // True for 12V voltage Compensation

```

### Getting encoder reads

For getting inner encoder reads you don't need to create a `RelativeEncoder` class, `ForgeSparkMax` already includes and configures the encoder, you can access its method and get an `EncoderResult` which you can add offsets, invert the read, put reductions, convertions or pass it directly to Radians / Degrees or RadiansPerSecond

``` java

    //Creation, ForgeSparkMax can assume you're using MotorType.kBrushless
    ForgeSparkMax driveSparkMax = new ForgeSparkMax(DriveConstants.frontLeft.DrivePort);
    ForgeSparkMax turnSparkMax = new ForgeSparkMax(DriveConstants.frontLeft.TurnPort);

    //Getting drive Velocity in Radians per Second
    double driveVelocity = driveSparkMax.getVelocity().toRadiansPerSecond().getRead();

    //Getting drive Position in Radians
    double drivePosition = driveSparkMax.getPosition().toRadians().getRead();

    //Getting Turn position with a reduction
    Rotation2d moduleAngle = 
    Rotation2d.fromRotations(
      turnSparkMax.getPosition().withReduction(turnMotorReduction).getRead());

```

### Commands

This motor class include the capability for easy voltage / speeds commands instead of creating a new command.

=== "Example One"

    ``` java 
      //Returning a Command for easy use outside this class
      public Command speedMotorCommand(double speed, Subsystem requirements){
        return m_motor.speedCommand(speed, requirements);
      } 
    ```

=== "Example Two"

    ``` java   
      //Returning a Command for easy use outside this class
      public Command speedMotorCommand(double speed){
        return m_motor.speedCommand(speed, this); //using this assuming this method is in an subsystem class 
      }  
    ```

### Publish

You can publish motor info to the Dashboard / Network tables through a `Sendable`

=== "SmartDashboard"

    ``` java 
      SmartDashboard.putData("DriveSpark", driveSparkMax); 
    ```

=== "NTPublisher"

    ``` java   
      NTPublisher.publish("SwerveModule", "Motors", driveSparkMax);  
    ```
    