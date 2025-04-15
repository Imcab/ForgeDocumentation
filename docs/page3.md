# SwerveLib - Forge

## Introduction

SwerveLib highlights the use of an own background updater pose estimator, easy built-in `SwerveWidget` and also a `poseFinder` with pathplanner.

!!! warning
    You **MUST** install PathPlannerLib for SwerveLib to work!

## PoseFinder

SwerveLib includes a more friendly `poseFinder` for pathFinding using [pathplanner's](https://pathplanner.dev/home.html) lib. It also supports parallel Command groups and other methods to use in your code.

### Creating a PoseFinder

For creating your PoseFinder you must pass a [field Object](/page6), a `PathConstraints` and other requirements.

``` java


    private PoseFinder pathFinder;

     private static final PathConstraints selectedConstraints = new PathConstraints(
        4.5,
        4.0,
        Units.degreesToRadians(540),
        Units.degreesToRadians(720)
    );

    public Swerve(){

        //PathPlanner's configuration must be done first!
        AutoBuilder.configure(
            this::getEstimatedPosition,
            this::setPose,
            this::getChassisSpeeds,
            this::runVelocity,
            new PPHolonomicDriveController(
                translationPPgains,
                rotationPPgains,
                0.02),
            getPathPlannerConfiguration(),
            ()-> DriverStation.getAlliance().orElse(Alliance.Blue) == Alliance.Red);


        pathFinder = new PoseFinder( 
            FieldObject.REEFSCAPE, //This year's field
            selectedConstraints, //The PathPlannerConstraints
            this::runVelocity, //Your method to move your swerve with a desired ChassisSpeeds
            this::getEstimatedPosition, //Your robot pose
            this::setPose, //Method to set your robot pose
            0.02, //Update period (you normally want to use this number)
            this); //Your swerve subsystem
    }

```

### Using PoseFinder

The best way to use your `PoseFinder` if you have it in your subsystem is to return it for it to be accessed in your `RobotContainer`

You should have a method like this in your swerve Subsystem.

``` java

    public PoseFinder getPathFinder(){
        return pathFinder;
    }

```

Now in the robot container you can access to the pathFinder and to all its method, **NOTE: All poses should be in the blue alliance reference, you shouldn't do any kind of flipping, poseFinder mirrors your pose to the Red Alliance automatically**

!!! warning
    All the poses are based on the blue alliance, poseFinder flips the pose if red alliance, like pathplanner flips their paths based on their alliance

``` java
    
    private void configureBindings() {
        //Going to a pose on button pressed (Blue Alliance Reference)
        driver.cross().whileTrue(chassis.getPathFinder().toPoseCommand(new Pose2d(2.5, 2.5, Rotation2d.kZero)));
 
    }

```

`PoseFinder` has conditional methods, parallel commands and deadline commands for you to use

``` java
    
    //Go to pose
    chassis.getPathFinder().toPoseCommand(new Pose2d(2.5, 2.5, Rotation2d.kZero));
    //Go to pose 1 or pose 2 based on x condition (supplier)
    chassis.getPathFinder().toIFPoseCommand(scoringPose, feederPose, ()-> hasPiece);
    //Go to the neareast Pose based on current robot's pose
    chassis.getPathFinder().toNearestPoseCommand(pose1,pose2,pose3,pose4);
    
```

All versions of these methods includes a parallel and a deadline version for you to compose commands.

### Follow Paths

`PoseFinder` has the hability to pathfind and then start a PathPlanner Path from pathName, to do this you can use the method `toPathPlannerPath` and you pass the path's name.

``` java
    
    //PathFind and the follow a path
    chassis.getPathFinder().toPathPlannerPath("MyPath");
    
```

!!! note
    This method is currently in beta so do it with precaution and be updated for future changes.

## ForgeSwerveDrivePoseEstimator

A wrapper around WPILib’s `SwerveDrivePoseEstimator` that runs pose estimation in the background, handles disconnected gyros, and optionally visualizes robot poses using Field2d. The benefits of running the estimation in the background is tha the odometer updates in parallel with the code and it does not need to wait for all previos line to update, getting faster and more reliable updates.

!!! warning
    Only one instance should be created. Running multiple instances will likely lead to incorrect pose estimation.

### Usage

For creating a `ForgeSwerveDrivePoseEstimator` you need to pass your robot's **kinematics**, a method to determine if your gyro is connected, your **module positions** your **gyro rotation** and a boolean for creating a field2D widget to show on dashboard. Optional: you can set up your custom update period, defaults 0.02 (Robot's default update cycles).

``` java
    
    public class Swerve{

        ForgeSwerveDrivePoseEstimator poseEstimator;

        public Swerve(){

            //Creates a new pose estimator
            poseEstimator = new ForgeSwerveDrivePoseEstimator(
                kinematics, //Your kinematics
                ()-> gyroConnection(), //Supplier to determine your gyro connection
                this::getModulePositions, //Your current SwerveModulePositions
                this::getnavXRotation, //Your gyro rotation2d
                true); //True for creating a widget on the dashboard
        }
    }
        
```

The usage is pretty similar as a traditional pose Estimator, the difference is that you don't need to call or worry for any update method because the estimator does it by itself, you can still get your robots pose, reset your robots pose and also get the fieldWidget for adding custom objects such as vision targets, or paths.

``` java
    
    public Pose2d getEstimatedPosition() {
        return poseEstimator.getEstimatedPose();
    }

    public void setPose(Pose2d pose) {
        poseEstimator.resetPosition(pose);
    }
        
```

## SwerveWidget

<img src="/assets/SW.png" alt="Logo" style="width:30%;">

Forge includes an easy setup for you to use the [Elastic's](https://frc-elastic.gitbook.io/docs/additional-features-and-references/widgets-list-and-properties-reference) **Swerve Custom Widget** through the `SwerveWidget` class

### Setting up a SwerveWidget

For setting up a swerve widget you must use the `SwerveModuleStateSupplier` class, which includes an angle and the module speed, both in doubles, you can create a widget using the `SwerveWidget` class with the method build:

First we create our moduleSuppliers, passing an velocity and our module rotation, this method should be done in the constructor!

``` java

    public Swerve(){

        SwerveModuleStateSupplier[] suppliers = new SwerveModuleStateSupplier[4];

        suppliers[0] = new SwerveModuleStateSupplier(
            ()-> modules[0].getModuleVelocity(),
            ()-> modules[0].getModuleRotation().getRadians());

        suppliers[1] = new SwerveModuleStateSupplier(
            ()-> modules[1].getModuleVelocity(),
            ()-> modules[1].getModuleRotation().getRadians());
            
        suppliers[2] = new SwerveModuleStateSupplier(
            ()-> modules[2].getModuleVelocity(),
            ()-> modules[2].getModuleRotation().getRadians());

        suppliers[3] = new SwerveModuleStateSupplier(
            ()-> modules[3].getModuleVelocity(),
            ()-> modules[3].getModuleRotation().getRadians());
    }

```

### Building

Now we can just use the `build` method to send it to the dashboard with a key or `buildCustomPath` to send it to a desired NetworkTables Path, for `buildCustomPath` to work you **must** have `NetworkTablesUtil` installed. Finnally we should pass the robots rotation in the desired unit, **must** be the same unit as the module angles.

#### Normal Build (Dashboard)

``` java

    public Swerve(){

        SwerveModuleStateSupplier[] suppliers = new SwerveModuleStateSupplier[4];

        suppliers[0] = new SwerveModuleStateSupplier(
            ()-> modules[0].getModuleVelocity(),
            ()-> modules[0].getModuleRotation().getRadians());

        suppliers[1] = new SwerveModuleStateSupplier(
            ()-> modules[1].getModuleVelocity(),
            ()-> modules[1].getModuleRotation().getRadians());
            
        suppliers[2] = new SwerveModuleStateSupplier(
            ()-> modules[2].getModuleVelocity(),
            ()-> modules[2].getModuleRotation().getRadians());

        suppliers[3] = new SwerveModuleStateSupplier(
            ()-> modules[3].getModuleVelocity(),
            ()-> modules[3].getModuleRotation().getRadians());

        SwerveWidget.build(
                "Elastic/SwerveWidget", //Your key in "String" to show on the SmartDashboard
                suppliers[0],
                suppliers[1],
                suppliers[2],
                suppliers[3],
                ()-> getRotation().getRadians() //Your robots rotation
        );
    }

```

#### Custom Build (NetworkTables)

``` java

    public Swerve(){

        SwerveModuleStateSupplier[] suppliers = new SwerveModuleStateSupplier[4];

        suppliers[0] = new SwerveModuleStateSupplier(
            ()-> modules[0].getModuleVelocity(),
            ()-> modules[0].getModuleRotation().getRadians());

        suppliers[1] = new SwerveModuleStateSupplier(
            ()-> modules[1].getModuleVelocity(),
            ()-> modules[1].getModuleRotation().getRadians());
            
        suppliers[2] = new SwerveModuleStateSupplier(
            ()-> modules[2].getModuleVelocity(),
            ()-> modules[2].getModuleRotation().getRadians());

        suppliers[3] = new SwerveModuleStateSupplier(
            ()-> modules[3].getModuleVelocity(),
            ()-> modules[3].getModuleRotation().getRadians());

        SwerveWidget.buildCustomPath(
            getTableKey(), //Your table key "String"
            "Elastic/SwerveWidget", //Your key "String"
            suppliers[0],
            suppliers[1],
            suppliers[2],
            suppliers[3],
            ()-> getRotation().getRadians() //Your robots rotation
        );
    }

```
