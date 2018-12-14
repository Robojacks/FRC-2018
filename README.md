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
