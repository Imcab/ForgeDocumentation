# Sim - Forge

## Introduction

Simulation feature in Forge comes with two custom annotations to hande simulation in your robot code

- `@RealDevice`: to handle real hardware devices / objects in your code
- `@SimulatedDevice`: to handle simulated objects in your code

These annotations are used to the code to decide which objects to create based if the robot is in simulation or not

## SimulatedSubsystem

For this annotations to work you must implement the `SimulatedSubsystem` interface.

``` java
    public class SwerveModule implements SimulatedSubsystem{
        ...
    }

```

### Using annotations

Now that your class has the `SimulatedSubsystem` implementation, you can declare your objects and add an one of two annotations if you want:

``` java

    public class SwerveModule implements SimulatedSubsystem{

        @RealDevice
        private CANcoder absoluteEncoder;

        @SimulatedDevice
        private DCMotorSim driveSim =
            new DCMotorSim(
            LinearSystemId.createDCMotorSystem(NEOGearbox, 0.025, driveMotorReduction),
            NEOGearbox);
    }

```

### Handling the annotations

After you declared your devices, you need to initializate the devices and passing a path for you to see your devices correctly in your constructor

``` java

    public class SwerveModule implements SimulatedSubsystem{

        @RealDevice
        private CANcoder absoluteEncoder;

        @SimulatedDevice
        private DCMotorSim driveSim =
            new DCMotorSim(
            LinearSystemId.createDCMotorSystem(NEOGearbox, 0.025, driveMotorReduction),
            NEOGearbox);

        public SwerveModule(int index){
   
            //Use real Configuration
            if (!isInSimulation()) {
                this.turnPIDGains = new PIDGains(5.0, 0,0);
                this.drivePIDGains = new PIDGains(0.05, 0, 0);
                this.driveFFGains = new SimpleFeedForwardGains(0.1, 0.08, 0);
            
                createSparks(index);

            }else{
            //Use sim Configuration
                this.turnPIDGains = new PIDGains(8.0, 0,0);
                this.drivePIDGains = new PIDGains(0.05, 0, 0);
                this.driveFFGains = new SimpleFeedForwardGains(0, 0.0789, 0);  
            }

            drivePID = new PIDControl(drivePIDGains);
            turnPID = new PIDControl(turnPIDGains);

            turnPID.continuousInput(-Math.PI, Math.PI);

            initializeSubsystemDevices("NetworkSwerve/Devices/Modules"); //Initializate after all your devices

        }
    }

```

### Subsystem Loops

Both, simulated and real devices must have its own loop, `SimulatedSubsystems` handles this two loops with its own functions, `RealDevicesPeriodic` and `SimulationDevicesPeriodic`

``` java
    
    @Override
    public void SimulationDevicesPeriodic(){

        this.moduleAngle = new Rotation2d(turnSim.getAngularPositionRad());
        this.driveVelocity = driveSim.getAngularVelocityRadPerSec();

        driveSim.setInputVoltage(MathUtil.clamp(driveVoltage, -12.0, 12.0));
        turnSim.setInputVoltage(MathUtil.clamp(turnVoltage, -12.0, 12.0));

        driveSim.update(0.02);
        turnSim.update(0.02);
    }

    @Override
    public void RealDevicesPeriodic(){

        this.moduleAngle = 
        absoluteEncoder.isConnected() ? 
            Rotation2d.fromRotations(absoluteEncoder.getAbsolutePosition().getValueAsDouble() - Offset) : 
            Rotation2d.fromRotations(turnSparkMax.getPosition().withReduction(turnMotorReduction).getRead());

        this.driveVelocity = driveSparkMax.getVelocity().toRadiansPerSecond().getRead();

        driveSparkMax.setVoltage(driveVoltage);
        turnSparkMax.setVoltage(turnVoltage);

    }

```

After you declared your own loops, you must call `handleSubsystemRealityLoop` to handle your devices in your **main** loop:

``` java

    public void periodic(){
        handleSubsystemRealityLoop(); // Call this method

        if (angleSetpoint != null) {
            this.turnVoltage = turnPID.calculate(moduleAngle.getRadians()).getOutput();

            if (speedSetpoint != null) {
                this.driveVoltage = drivePID.calculate(driveVelocity).plus(()-> ffVolts).getOutput();
            }else{
                drivePID.reset();
            }
        }else{
            turnPID.reset();
        }
    }
    
    @Override
    public void SimulationDevicesPeriodic(){

        this.moduleAngle = new Rotation2d(turnSim.getAngularPositionRad());
        this.driveVelocity = driveSim.getAngularVelocityRadPerSec();

        driveSim.setInputVoltage(MathUtil.clamp(driveVoltage, -12.0, 12.0));
        turnSim.setInputVoltage(MathUtil.clamp(turnVoltage, -12.0, 12.0));

        driveSim.update(0.02);
        turnSim.update(0.02);
    }

    @Override
    public void RealDevicesPeriodic(){

        this.moduleAngle = 
        absoluteEncoder.isConnected() ? 
            Rotation2d.fromRotations(absoluteEncoder.getAbsolutePosition().getValueAsDouble() - Offset) : 
            Rotation2d.fromRotations(turnSparkMax.getPosition().withReduction(turnMotorReduction).getRead());

        this.driveVelocity = driveSparkMax.getVelocity().toRadiansPerSecond().getRead();

        driveSparkMax.setVoltage(driveVoltage);
        turnSparkMax.setVoltage(turnVoltage);

    }

```

### isInSimulation

Implementing this interface adds you `isInSimulation` a method to check if your robot code is running in simulation

Example usage:

``` java
    //Use real Configuration
        if (!isInSimulation()) {
            this.turnPIDGains = new PIDGains(5.0, 0,0);
            this.drivePIDGains = new PIDGains(0.05, 0, 0);
            this.driveFFGains = new SimpleFeedForwardGains(0.1, 0.08, 0);
            
            createSparks(index);

        }else{
        //Use sim Configuration
            this.turnPIDGains = new PIDGains(8.0, 0,0);
            this.drivePIDGains = new PIDGains(0.05, 0, 0);
            this.driveFFGains = new SimpleFeedForwardGains(0, 0.0789, 0);

            
        }

```
