Robot.java
    
    package org.usfirst.frc.team4674.robot;

    import edu.wpi.cscore.UsbCamera;
    import edu.wpi.first.wpilibj.CameraServer;
    import edu.wpi.first.wpilibj.TimedRobot;
    import edu.wpi.first.wpilibj.Timer;
    import edu.wpi.first.wpilibj.smartdashboard.SendableChooser;
    import edu.wpi.first.wpilibj.smartdashboard.SmartDashboard;

    /**
    * The VM is configured to automatically run this class, and to call the
    * functions corresponding to each mode, as described in the IterativeRobot
    * documentation. If you change the name of this class or the package after
    * creating this project, you must also update the build.properties file in the
    * project.
    */
    public class Robot extends TimedRobot {
	   private static final String kDefaultAuto = "Default";
	   private static final String kCustomAuto = "My Auto";
	   private String m_autoSelected;
	   private SendableChooser<String> m_chooser = new SendableChooser<>();
	   private Timer AutoTimer;

	TankDrive chassis = new TankDrive(); // added this (1), getting an instance of the tank drive class made in the other file in this project

	/**
	 * This function is run when the robot is first started up and should be
	 * used for any initialization code.
	 */
	@Override
	public void robotInit() {
		m_chooser.addDefault("Default Auto", kDefaultAuto);
		m_chooser.addObject("My Auto", kCustomAuto);
		SmartDashboard.putData("Auto choices", m_chooser);

		chassis.init(); // added this (2), initializing joystick controllers, motors, speed controller groups, and differential drive.

		UsbCamera camera0 = CameraServer.getInstance().startAutomaticCapture(0);
		UsbCamera camera1 = CameraServer.getInstance().startAutomaticCapture(1);
		// Create timer for auto
		AutoTimer= new Timer();
	}

	/**
	 * This autonomous (along with the chooser code above) shows how to select
	 * between different autonomous modes using the dashboard. The sendable
	 * chooser code works with the Java SmartDashboard. If you prefer the
	 * LabVIEW Dashboard, remove all of the chooser code and uncomment the
	 * getString line to get the auto name from the text box below the Gyro
	 *
	 * <p>You can add additional auto modes by adding additional comparisons to
	 * the switch structure below with additional strings. If using the
	 * SendableChooser make sure to add them to the chooser code above as well.
	 */
	@Override
	public void autonomousInit() {
		//		m_autoSelected = m_chooser.getSelected();
		// m_autoSelected = SmartDashboard.getString("Auto Selector",
		// 		kDefaultAuto);
		//		System.out.println("Auto selected: " + m_autoSelected);
		AutoTimer.reset();
		AutoTimer.start();
	}

	/**
	 * This function is called periodically during autonomous.
	 */
	@Override
	public void autonomousPeriodic() {
		if (AutoTimer.get() <= 6) {

			chassis.roboDrive.tankDrive(-0.5, -0.5);

		} else {
			
			chassis.roboDrive.tankDrive(0, 0);
			
			AutoTimer.stop();
		}
		/*switch (m_autoSelected) {
			case kCustomAuto:
				chassis.auto();
				break;
			case kDefaultAuto:
			default:
				// Put default auto code here
				break;
		}  commented out */

	}

	/**
	 * This function is called periodically during operator control.
	 */
	@Override
	public void teleopPeriodic() {
		chassis.run(); // added this (3). running safety features and accessing values in xbox controller with individual joysticks on it.
	}

	/**
	 * This function is called periodically during test mode.
	 */
	@Override
	public void testPeriodic() {
	}
    }

TankDrive.java    
    
    package org.usfirst.frc.team4674.robot;

    //imports
    
    import edu.wpi.first.wpilibj.XboxController;

    //import edu.wpi.first.wpilibj.Joystick;
    
    import edu.wpi.first.wpilibj.SpeedControllerGroup;
    
    import edu.wpi.first.wpilibj.Timer;
    
    import com.ctre.phoenix.motorcontrol.can.WPI_TalonSRX;
    
    import edu.wpi.first.wpilibj.drive.DifferentialDrive;
    
    import edu.wpi.first.wpilibj.Compressor;

    import edu.wpi.first.wpilibj.Solenoid;

    public class TankDrive {
	
	// instance variables
	
	XboxController xbox;
	
	WPI_TalonSRX RRearWheel; // Defining all eight motors
	WPI_TalonSRX RFrontWheel;
	WPI_TalonSRX LRearWheel;
	WPI_TalonSRX LFrontWheel;
	WPI_TalonSRX RArm;
	WPI_TalonSRX LArm;
	WPI_TalonSRX RIntake;
	WPI_TalonSRX LIntake;
	Compressor Cramp;
	Solenoid Valve1;
	Solenoid Valve2;
	
	SpeedControllerGroup left; // Defining controllers of certain sides of motors
	SpeedControllerGroup right;
	
	SpeedControllerGroup Arm; // moves both motors that controls the arm.
	
	SpeedControllerGroup Collector;
	
	DifferentialDrive roboDrive; // Defines a drive that controls both speed controllers
	
	double speedChk; // Defines variable that changes the sensitivity of the robot controls in roboDrive during the run() method.
	
	boolean flag = true;
	
	boolean btnChk = true;
	
	// methods
	
	public void init() {
		
		xbox = new XboxController(3);
		
		RRearWheel = new WPI_TalonSRX(0); // right rear wheel
		RFrontWheel = new WPI_TalonSRX(1); // right front wheel
		LRearWheel = new WPI_TalonSRX(2); // left rear wheel
		LFrontWheel = new WPI_TalonSRX(3); // left front wheel
		RArm = new WPI_TalonSRX(4); // right arm
		LArm = new WPI_TalonSRX(5); // left arm
		RIntake = new WPI_TalonSRX(6); // right intake
		LIntake = new WPI_TalonSRX(7); // left intake
		Cramp = new Compressor(20);
		Valve1 = new Solenoid(20, 0);
		Valve2 = new Solenoid(20, 1);
		
		Arm = new SpeedControllerGroup(LArm, RArm);
		
		Collector = new SpeedControllerGroup(LIntake, RIntake);
		
		right = new SpeedControllerGroup(RRearWheel, RFrontWheel); // right speed controller group 
		left = new SpeedControllerGroup(LRearWheel, LFrontWheel); // left speed controller group 
		roboDrive = new DifferentialDrive(left, right); // making both speed controllers part of the overall drive
		
		speedChk = 0.5; // setting sensitivity of robot controls. Do not go lower than 0.3 since it won't register on the robot
		
		btnChk = true;
		
		flag = true;
		
		Cramp.start();
		
	}
	
	public void run() {
		roboDrive.setSafetyEnabled(true); // stops robot if it loses communication
		
		// drive robot
		
		roboDrive.tankDrive(xbox.getRawAxis(1), xbox.getRawAxis(3));
		
		// Compressor
		
		if (xbox.getRawButton(5)) { // compress
			
			Valve1.set(false);
			
			Valve2.set(false);
			
		} else if (xbox.getRawButton(6)) { // uncompress
			
			Valve1.set(true);
			
			Valve2.set(true);
		}
		
		// move arm
		if (xbox.getRawButton(7)) {
			Arm.set(0.7);
			
		} else if (xbox.getRawButton(8)) {
			Arm.set(-0.7);
			
		} else {Arm.set(0);}
		
		if (xbox.getRawButton(10)) { // send it
			
			Arm.set(-1);
			
			btnChk = true;
			
		} else if (btnChk) {
			Arm.set(0);
			
			btnChk = false;
			
		}
		
		
		// Separate Drive
		
		if (xbox.getRawButton(1)) { // take in cube left
			
			LIntake.set(-0.45);
			
		} else if (xbox.getRawButton(3)) { // push out cube left
			
			LIntake.set(0.45); 
		
		} else {LIntake.set(0);}
		
		if (xbox.getRawButton(2)) { // take out cube right
			
			RIntake.set(-0.45);
			
		} else if (xbox.getRawButton(4)) { // take in cube right
				
			RIntake.set(0.45);
				
		} else {RIntake.set(0);}
		
		// Double Intake
		
		if (xbox.getRawButton(11)) { // Intake
			
			Collector.set(-0.45);
			
			btnChk = true;
			
		} else if (xbox.getRawButton(12)) { // Outtake
			
			Collector.set(0.45);
			
			btnChk = true;
			
		} else if (btnChk) {
			
			Collector.set(0);
			
			btnChk = false;
		}
		
		 
	}
	
	public void auto() {
		
		roboDrive.setSafetyEnabled(false);
		
		if (flag) {
		
		// Drive forward
		roboDrive.tankDrive(0.5, 0.5);
		
		// stop
		roboDrive.tankDrive(0, 0);
		
		System.out.println("end of program");
		
		flag = false;
		
		}
	}
	
    }

