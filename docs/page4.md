# RobotState - Forge

## Introduction

The `RobotState` class allows for users to interact with the diferent robot modes through an interface `RobotLifeCycle` without putting their logic directly into de `Robot` class, currently it supports these robot modes:

- teleopInit
- teleopPeriodic
- teleopExit
- autonomousInit
- autonomousPeriodic
- disabledInit
- disabledPeriodic
- disabledExit
- autonomousExit

## Using RobotLifeCycle

### Class

!!! info
    If you create your forge Project through [new projects](/docs/page1.md#new-projects), you don't need to
    manually implement it to your `Robot` class!

To use `RobotLifeCycle`, first you need to implement it to your class:

``` java
    //Using the robotLifeCycle
    public class ExampleClass implements RobotLifeCycle{
    
        public ExampleClass(){}
    
    }

```

#### Methods

Now that you have `RobotLifeCycle` in your class, you can now **@Override** the methods you want to use!

``` java
    //Using the robotLifeCycle
    public class ExampleClass implements RobotLifeCycle{
    
        public ExampleClass(){}

        @Override
        public void teleopInit(){
            System.out.println("Initializing teleop...");
        }


        @Override
        public void teleopPeriodic(){
            System.out.println("Teleop!!!!");
        }

}

```

## Next Steps

### Robot Container

Now for that you implemented your logic, in the `Robot Container` or in the class you have all your subsystems,
you **need to create a list of all the subystems** that implements the `RobotLyfeCycle` and returning it here is an example below:

Creating the list:

``` java
    //Creating the list
    private final List<RobotLifeCycle> lifecycleSubsystems;

```

Returning the list:

``` java
    //Returning the subsystems
    public List<RobotLifeCycle> getLifeCycle() {
        return lifecycleSubsystems;
    }

```

An implementation in `RobotContainer`:

``` java

    public class RobotContainer {
    
    //Subsystems that implement RobotLifeCycle
    private final NetworkSwerve chassis;

    //Subsystems that implement RobotLifeCycle
    private final ExampleClass exampleClass;

    //Creating the list
    private final List<RobotLifeCycle> lifecycleSubsystems;

    public RobotContainer() {

        chassis = new NetworkSwerve(SwervePathConstraints.kNormal);

        exampleClass = new ExampleClass();

        lifecycleSubsystems = List.of(RobotState.getInstance(), exampleClass); //Add more subsystems here that implements RobotLifeCycle class

        configureBindings();
    }

    private void configureBindings() {}

    public Command getAutonomousCommand() {
        return Commands.print("No autonomous command configured");
    }

    //Returning the subsystems
    public List<RobotLifeCycle> getLifeCycle() {
        return lifecycleSubsystems;
    }
}

```

!!! warning
    You need to implement all of your subsystems and you need yo have the list in `RobotContainer` for
    `RobotLifeCycle` to work!

### Robot

Finally after you get all your setUp in the `RobotContainer` you must put an additional lines in the `Robot` file.

You can just copy this template:

``` java
    // Copyright (c) FIRST and other WPILib contributors.
    // Open Source Software; you can modify and/or share it under the terms of
    // the WPILib BSD license file in the root directory of this project.

    package frc.robot;

    import edu.wpi.first.wpilibj.TimedRobot;
    import edu.wpi.first.wpilibj2.command.Command;
    import edu.wpi.first.wpilibj2.command.CommandScheduler;
    import lib.Forge.RobotState.RobotLifeCycle;

    public class Robot extends TimedRobot {

    private Command m_autonomousCommand;

    private final RobotContainer m_robotContainer;

    public Robot() {

        m_robotContainer = new RobotContainer();

    }

    @Override
    public void robotPeriodic() {
        CommandScheduler.getInstance().run();

    }

    @Override
    public void disabledInit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::disabledInit);
    }

    @Override
    public void disabledPeriodic() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::disabledPeriodic);
    }

    @Override
    public void disabledExit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::disabledExit);
    }

    @Override
    public void autonomousInit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::autonomousInit);
        m_autonomousCommand = m_robotContainer.getAutonomousCommand();

        if (m_autonomousCommand != null) {
            m_autonomousCommand.schedule();
        }
    }

    @Override
    public void autonomousPeriodic() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::autonomousPeriodic);
    }

    @Override
    public void autonomousExit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::autonomousExit);
    }

    @Override
    public void teleopInit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::teleopInit);
        if (m_autonomousCommand != null) {
            m_autonomousCommand.cancel();
        }
    }

    @Override
    public void teleopPeriodic() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::teleopPeriodic);
    }

    @Override
    public void teleopExit() {
        m_robotContainer.getLifeCycle().forEach(RobotLifeCycle::teleopExit);
    }

    @Override
    public void testInit() {
        CommandScheduler.getInstance().cancelAll();
    }

    @Override
    public void testPeriodic() {}

    @Override
    public void testExit() {}
    }


```
