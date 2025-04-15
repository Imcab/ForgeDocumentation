# NetworkTablesUtil - Forge

## NTPublisher

`NTPublisher` is the ultimate publisher through NetworkTables, works similar and better as the traditional `SmartDashboard` and also has the hability to retreive more data types

| Features                               |        NTPublisher       | SmartDashboard         |
| ---------------------------------------| -------------------------| -----------------------|
| Publishing normal values               |             ✅          |            ✅          |
| Publishing special values (Pose2d, etc)|             ✅          |            ❌          |
| Over 30 supported data types           |             ✅          |            ❌          |
| Retrieve over 30 data types            |             ✅          |            ❌          |
|  Sendable support                      |             ✅          |            ✅          |
| Different data paths                   |             ✅          |            ❌          |

### Publishing values

`NTPublisher` works the same as `SmartDashboard` with the extra step of you passing the table you want to publish your data, `SmartDashboard` puts you data in **SmartDashboard/YourData** but `NTPublisher` lets you custom your directory for better organization like: **MyDirectory/MyData**. Also `NTPublisher` lets you put your data un subfolders as: **MyDirectory/MySubFolder/MyData**.

#### Example

``` java
    NTPublisher.publish("Robot", "GyroConnection", gyro.isConnected());
```

#### SubFolder example

``` java
    NTPublisher.publish("Robot", "Odometry/Pose", poseEstimator.getEstimatedPose());
```

### Retrieving values

For retrieving values you just specify the directory and the key where the data is located and provide a default value in case the data is not found

``` java
    //Gets the gyro connection
    // and return false in case NTPublisher fails in locating the value
    boolean connection = NTPublisher.retrieve("Robot", "GyroConnection", false); 
```

``` java
    //Returns 0,0,new Rotation2d() for default value in case of failure
    Pose2d botPose = NTPublisher.retrieve("Robot", "Odometry/Pose", Pose2d.kZero);
```

### Supported Data Types

`NTPublisher` supports:

- `double`
- `double[]`
- `boolean`
- `boolean[]`
- `String`
- `String[]`
- `ChassisSpeeds`
- `ChassisSpeeds[]`
- `Pose2d`
- `Pose2d[]`
- `Pose3d`
- `Pose3d[]`
- `Rotation2d`
- `Rotation2d[]`
- `Rotation3d`
- `Rotation3d[]`
- `Transform2d`
- `Transform2d[]`
- `Transform3d`
- `Transform3d[]`
- `Translation2d`
- `Translation2d[]`
- `Translation3d`
- `Translation3d[]`
- `SwerveModuleState`
- `SwerveModuleState[]`
- `SwerveModulePosition`
- `SwerveModulePosition[]`
- `Color`
- `Sendable`

### Manual Sendable Update

For `NTPublisher` to update the `Sendables` you must put an extra line in your `robot` file or in any method that update periodically, you **must** call the method above

``` java
    NTPublisher.updateAllSendables();
```

This method is **already included** in the `Forge templates` and you don't need to do it unless you manually install Forge

!!! note
    This section is for users that install Forge manually and not through a template!

## NetworkSubsystem

`NetworkSubsystem` a wrapper class for `SubsystemBase` comes with various features such as:

- AutoPublish values
- Autopublish commands
- Own subsystem directory
- Easy use of `NTPublisher`
- Display subsystem information

### Extending NetworkSubsystem

For using `NetworkSubsystem` you need to extend it from your subsystem class instead of `SubsystemBase`, don't worry, `NetworkSubsystem` already extends `SubsystemBase` for you to use

``` java

    public class ExampleSubsystem extends NetworkSubsystem{
        ...
    }
```

Now, after you extend the `NetworkSubsystem` class, you must implement some methods and call the constructor of the `NetworkSubsystem` class which needs an String as the subsystem's directory and a boolen indicating wether to show or now subsystem information

``` java

    public class ExampleSubsystem extends NetworkSubsystem{

        //Constructor with the NTSubsystem parameters
        public ExampleSubsystem(){
            super("AwesomeSubsystem", true); //Always make sure this line is called first
        }

        //Use this method as the normal "periodic" method, DO NOT OVERRIDE the periodic() method
        //As it handles internal logic and updates
        @Override
        public void NetworkPeriodic(){}
    }
```

!!! failure
    **Don't** override the default periodic() method, use NetworkPeriodic() instead!

### Using NetworkSubsystem

Now that you setup your subsystem you can use various features that this class provides!

#### AutoPublish

If you want to auto publish a method, you can use the `@AutoNetworkPublisher` annotation, provide a key and your value is publisher automatically!

``` java

    //This will be published to YourSubsystemKey/Modules/ChassisSpeeds
    @AutoNetworkPublisher(key = "Modules/ChassisSpeeds")
    public ChassisSpeeds getChassisSpeeds(){
        return kinematics.toChassisSpeeds(getModuleStates());
    }

```

!!! note
    For AutoNetworkPublisher to work, it must be a compatible data type from `NTPublisher`

#### PublishCommands

You can publish a Command to dashboards and have a button to trigger the command on the dashboard, just use the annotation `@NetworkCommand` and pass a desired key!

``` java

    @NetworkCommand("Commands/ResetHeading")
    public Command resetHeadingCommand(){
        return Commands.runOnce(()-> resetHeading(), this);
    }

```

#### Publish and retrieving data

`NetworkSubsystem` already provides you the table for you to publish values with `NTPublisher`

Publishing values, Instead of doing:

``` java

    NTPublisher.publish("MySubsystem", "Modules/Locations", getModuleLocations());

```

You can just call:

``` java
    publishOutput("Modules/Locations", getModuleLocations());

```

And it will work exactly the same! all data types supported by `NTPublisher` are supported by `NetworkSubsystem`

And for retrieving a value just use:

``` java
    
    //False is the default value we want to use in case
    //The path is not found
    boolean myValue = getOutput("AwesomeValue", false);

```

## NTListener

`NTListener<T>` is a generic utility class for subscribing to real-time data updates from WPILib's NetworkTables. It enables tracking of value changes on a specific key in a specific table, supporting a wide range of data types including basic primitives, arrays, and WPILib geometry/kinematics classes.

### Using Listener

You must use the provided static factory methods to instantiate an NTListener. Each method creates a listener for a specific data type:

Primitives

``` java

    NTListener<Double> ofDouble(String tableName, String key);
    NTListener<Boolean> ofBoolean(String tableName, String key);
    NTListener<String> ofString(String tableName, String key);
```

GeometryTypes

``` java

    NTListener<Pose2d> ofPose2d(...);
    NTListener<Pose3d> ofPose3d(...);
    NTListener<Translation2d> ofTranslation2d(...);
    NTListener<Translation3d> ofTranslation3d(...);
    NTListener<Rotation2d> ofRotation2d(...);
    NTListener<Rotation3d> ofRotation3d(...);
    NTListener<Transform2d> ofTransform2d(...);
    NTListener<Transform3d> ofTransform3d(...);

```

Kinematics

``` java

    NTListener<SwerveModuleState> ofSwerveModuleState(...);
    NTListener<SwerveModulePosition> ofSwerveModulePosition(...);
    NTListener<ChassisSpeeds> ofChassisSpeeds(...);
```

Color

``` java

    NTListener<Color> ofColor(...);
```

All of the above types also support array-based listeners:

``` java

    NTListener<double[]> ofDoubleArray(...);
    NTListener<Pose2d[]> ofPose2dArray(...);
    //... and so on
```

Example of a `NTListener` creation

``` java

    NTListener<Pose2d> robotPoseListener = NTListener.ofPose2d("Vision", "RobotPose");
```

- Initializes a listener on a NetworkTable entry.

- Subscribes to kValueAll events.

- Automatically tracks value changes.

### Methods

boolean `hasChanged()`

``` java

    if(robotPoseListener.hasChanged()){
        //Do something if pose changes
    }

```

Returns true if the value has changed since the last call.

Resets internal change flag after read.

T `getValue()`

``` java

    if (robotPoseListener.hasChanged()) {
        Pose2d updatedPose = robotPoseListener.getValue(); //Get the lastest Pose
    }

```

Retrieves the current value from NetworkTables.

Uses the appropriate NTPublisher.retrieve method for type safety.

!!! info
    This listener does not handle custom object serialization—only types supported by NTPublisher.

!!! warning
    Make sure to register the listener only once per key to avoid duplicates.

## NTTunnableNumber

`NTTunnableNumber` is a utility class designed to simplify the management of tunable numerical values using NetworkTables. It allows for real-time tuning and monitoring of double values from both the robot code and external interfaces like dashboards.

!!! info
    Be sure to call update() periodically to keep the internal value in sync with NetworkTables.

Features include:

- Subscribe to and publish double values via NetworkTables

- Track and modify values in real time

- Publish only when values change

- Detect whether a value has changed externally

### Using Tunnable number

Parameters for creating a `NTTunnableNumber` are:

key: The NetworkTables key for the value

defaultValue: The initial value if no value exists in NetworkTables

#### Example constructor

``` java

    NTTunnableNumber maxSpeed = new NTTunnableNumber("Drive/MaxSpeed", 3.0);

```

### Methods

`update()`

Updates the internal stored value from the latest subscriber value. Must be called periodically (e.g., in subsystem periodic() method).

``` java

    maxSpeed.update();

```

`get()`

Returns the current value stored locally (after calling update()).

``` java

    double speed = maxSpeed.get();

```

`set(double value)`

Sets a new value and publishes it to NetworkTables. Only publishes if the value has changed.

``` java

    maxSpeed.set(4.0);
```

`hasChanged()`

Checks if the value from NetworkTables is different from the last value set via set() or update().

``` java

    if (maxSpeed.hasChanged()) {
        System.out.println("Value changed externally!");
    }
```

Example integration:

``` java

    public class DriveSubsystem extends SubsystemBase {

        private final NTTunnableNumber maxSpeed = new NTTunnableNumber("Drive/MaxSpeed", 3.0);

        @Override
        public void periodic() {
            maxSpeed.update();
            SmartDashboard.putNumber("Current Max Speed", maxSpeed.get());
        }
    }
```

#### Best Practices

- Always call update() before using get() to ensure fresh values

- Use consistent NetworkTables paths for tuning parameters

- Group related keys under the same subsystem namespace (e.g., "Shooter/PID/kP")

## NTJoystick

`NTJoystick` supports for easy logging of joysticks without having advantage kit, it logs all the values of the avis, buttons, povs and special buttons. Currently supports `CommandXBoxController`, `CommandPS4Controller`, `CommandPS5Controller` and you can manually send your joystick values if you don't use the classes from above.

### Usage

This class works with `NTPublisher` but can also work with `SmartDashboard`, all you have to do is call the static method `NTJoystick.from(yourJoystick)` and pass your joystick and thats it!

=== "NTPublisher"

    ``` java
     
        public class RobotContainer {

            private final CommandPS5Controller driver = new CommandPS5Controller(0);

            public RobotContainer() {
  
                NTPublisher.publish("NTControllers", "Driver1", NTJoystick.from(driver)); //publish driver joystick

                configureBindings();
            }
        } 
    ```

=== "SmartDashboard"

    ``` java
        
        public class RobotContainer {

            private final CommandPS5Controller driver = new CommandPS5Controller(0);

            public RobotContainer() {
  
                SmartDashboard.putData("Driver1", NTJoystick.from(driver)); //publish driver joystick to SmartDashboard

                configureBindings();
            }
        }
    ```


## NTSwerve

Publishes swerve drive data to NetworkTables.

- ChassisSpeeds – linear & angular robot velocity

- Rotation2d – robot orientation

- SwerveModuleState[] – states of all swerve modules

### Using NTSwerve

``` java

    NTSwerve ntSwerve = new NTSwerve("/MySwerve");

    ntSwerve.sendSwerve(chassisSpeeds, rotation, moduleStates);

```

Efficient for visualizing robot motion and module states in real time.

## NTCommand

Extends `Command` and auto-publishes it to NetworkTables on creation.

### Using NTCommand

``` java

    public class MyCommand extends NTCommand {

        public MyCommand() {
            super("/MyCommands", "DriveCommand");
        }
    }
```

Publishes command state (running, finished, etc.) for visualization/debugging.

## NTPoseEstimator

Publishes poses from a SwerveDrivePoseEstimator to NetworkTables using multiple slots. (In case of debugging multiple `SwerveDrivePoseEstimator`).

### Using NTPoseEstimator

``` java

    NTPoseEstimator posePublisher = new NTPoseEstimator("/Pose/", 3);
    posePublisher.sendEstimator(odometer, 0);
```

Each slot stores the robot's estimated pose for modular multi-robot or multi-instance dashboards.

## NTArrayBoolean

Manually publishes boolean arrays to NetworkTables.

``` java

    NTArrayBoolean boolPub = new NTArrayBoolean("MyKey");
    boolPub.sendBoolean(new boolean[] { true, false });
```

## NTArrayData<T>

Manually publishes arrays of `struct`-based data to NetworkTables.

``` java

    NTArrayData<MyStructType> dataPub = new NTArrayData<>("Key", MyStructType.struct);
    dataPub.sendData(new MyStructType[] { ... });

```

## NTArrayDouble

Manually publishes double[] to NetworkTables.

``` java

    NTArrayDouble doublePub = new NTArrayDouble("Key");
    doublePub.sendDouble(new double[] { 1.0, 2.0, 3.0 });

```

## NTArrayString

Manually publishes String[] to NetworkTables.

``` java

    NTArrayString stringPub = new NTArrayString("Key");
    stringPub.sendString(new String[] { "hello", "world" });

```

## NTBoolean

Manually publishes a single boolean value to NetworkTables.

``` java

    NTBoolean boolPub = new NTBoolean("Key");
    boolPub.sendBoolean(true);

```

## NTData<T>

Manually publishes a single `struct`-based data object.

``` java

    NTData<MyStruct> dataPub = new NTData<>("Key", MyStruct.struct);
    dataPub.sendData(myStructInstance);

```

## NTDouble

Manually publishes a single double value to NetworkTables.

``` java

    NTDouble doublePub = new NTDouble("Key");
    doublePub.sendDouble(3.14);

```

## NTString

Manually publishes a single String to NetworkTables.

``` java

    NTString stringPub = new NTString("Key");
    stringPub.sendString("Hello, world!");

```

## NTSupplierBoolean

Manually publishes a boolean from a `Supplier<Boolean>.`

``` java

    NTSupplierBoolean boolPub = new NTSupplierBoolean("Key", () -> myBoolean);
    boolPub.update();

```

## NTSupplierBooleanArray

Manually publishes a Boolean[] from a `Supplier<Boolean[]>.`

``` java

    NTSupplierBooleanArray arrayPub = new NTSupplierBooleanArray("Key", () -> myArray);
    arrayPub.update();

```

## NTSupplierDouble

Manually publishes a double from a `Supplier<Double>.`

``` java

    NTSupplierDouble doublePub = new NTSupplierDouble("Key", () -> 3.14);
    doublePub.update();

```

## NTSupplierDoubleArray

Manually publishes a double[] from a `Supplier<double[]>.`

``` java

    NTSupplierDoubleArray arrayPub = new NTSupplierDoubleArray("Key", () -> new double[] {1.0, 2.0});
    arrayPub.update();

```
