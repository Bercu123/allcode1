package org.firstinspires.ftc.teamcode;

import android.app.Activity;
import android.graphics.Color;
import android.view.View;

import com.qualcomm.robotcore.eventloop.opmode.LinearOpMode;
import com.qualcomm.robotcore.eventloop.opmode.TeleOp;
import com.qualcomm.robotcore.hardware.ColorSensor;
import com.qualcomm.robotcore.hardware.DcMotorEx;
import com.qualcomm.robotcore.hardware.TouchSensor;
import com.qualcomm.robotcore.util.ElapsedTime;
import com.qualcomm.robotcore.util.Range;

@TeleOp(name="Basic: teste  ", group="teste ")

public class teste extends LinearOpMode {

    // Declare OpMode members.
    private ElapsedTime runtime = new ElapsedTime();
    private DcMotorEx frontLDrive = null;
    private DcMotorEx backLDrive = null;
    private DcMotorEx frontRDrive = null;
    private DcMotorEx backRDrive = null;
    private DcMotorEx dreaptaMDrive = null;
    private DcMotorEx stangaMDrive = null;
    private DcMotorEx perieDrive = null;
    private TouchSensor touchSensor;
    private ColorSensor colorSensor;

    @Override

    public void runOpMode() {
        telemetry.addData("Status", "Initialized");
        telemetry.update();
        frontLDrive = hardwareMap.get(DcMotorEx.class, "frontLDrive");
        frontRDrive = hardwareMap.get(DcMotorEx.class, "frontRDrive");
        backLDrive = hardwareMap.get(DcMotorEx.class, "backLDrive");
        backRDrive = hardwareMap.get(DcMotorEx.class, "backRDrive");
        dreaptaMDrive = hardwareMap.get(DcMotorEx.class, "dreaptaMDrive");
        stangaMDrive = hardwareMap.get(DcMotorEx.class, "stangaMDrive");
        touchSensor = hardwareMap.get(TouchSensor.class, "touchSensor");
        perieDrive = hardwareMap.get(DcMotorEx.class, "perieDrive");
        colorSensor = hardwareMap.get(ColorSensor.class, "sensor_color");
        float hsvValues[] = {0F, 0F, 0F};
        final float values[] = hsvValues;
        int relativeLayoutId = hardwareMap.appContext.getResources().getIdentifier("RelativeLayout", "id", hardwareMap.appContext.getPackageName());
        final View relativeLayout = ((Activity) hardwareMap.appContext).findViewById(relativeLayoutId);

        frontLDrive.setDirection(DcMotorEx.Direction.REVERSE);
        backLDrive.setDirection(DcMotorEx.Direction.REVERSE);
        frontRDrive.setDirection(DcMotorEx.Direction.FORWARD);
        backRDrive.setDirection(DcMotorEx.Direction.FORWARD);
        dreaptaMDrive.setDirection(DcMotorEx.Direction.FORWARD);
        stangaMDrive.setDirection(DcMotorEx.Direction.FORWARD);
        perieDrive.setDirection(DcMotorEx.Direction.FORWARD);


        waitForStart();
        runtime.reset();


        while (opModeIsActive()) {

            double frontleftPower;
            double frontrightPower;
            double backleftPower;
            double backrightPower;
            double dreaptaMPower;
            double stangaMPower;
            double periePower;
            boolean bPrevState = false;
            boolean bCurrState = true;
            boolean bLedOn = true;
            colorSensor.enableLed(bLedOn);

            double inainte = gamepad1.right_trigger;
            double inapoi = gamepad1.left_trigger;
            double perie = -gamepad1.left_stick_y;
            bCurrState = gamepad1.x;


            stangaMPower = Range.clip(inainte, 0, 0.5);
            stangaMPower = Range.clip(inapoi, -0.5, 0);
            dreaptaMPower = Range.clip(inainte, 0, 0.5);
            dreaptaMPower = Range.clip(inapoi, -0.50, 0);
            periePower = Range.clip(perie, -0.5, 0);


            stangaMDrive.setPower(stangaMPower);
            dreaptaMDrive.setPower(dreaptaMPower);
            perieDrive.setPower(periePower);
            double max;

            // POV Mode uses left joystick to go forward & strafe, and right joystick to rotate.
            double axial = gamepad1.left_stick_x;  // Note: pushing stick forward gives negative value
            double lateral = -gamepad1.left_stick_y;
            double yaw = gamepad1.right_stick_x;

            // Combine the joystick requests for each axis-motion to determine each wheel's power.
            double leftFrontPower = (axial + lateral + yaw);
            double rightFrontPower = (axial - lateral - yaw);
            double leftBackPower = (axial - lateral + yaw);
            double rightBackPower = (axial + lateral - yaw);


            // Normalize the values so no wheel power exceeds 100%
            max = Math.max(Math.abs(leftFrontPower), Math.abs(rightFrontPower));
            max = Math.max(max, Math.abs(leftBackPower));
            max = Math.max(max, Math.abs(rightBackPower));

            if (max > 1.0) {
                leftFrontPower /= max;
                rightFrontPower /= max;
                leftBackPower /= max;
                rightBackPower /= max;
            }

            // Send calculated power to wheels
            if (touchSensor.isPressed()) {
                //frontleftPower=-1;
                //  frontrightPower=-1;
                // backrightPower=-1;
                // backleftPower=-1;
                telemetry.addData("Touch Sensor", touchSensor.isPressed());
                frontLDrive.setPower(-1);
                frontRDrive.setPower(1);
                backLDrive.setPower(1);
                backRDrive.setPower(-1);
                sleep(Long.parseLong("500"));
                if (bCurrState && (bCurrState != bPrevState)) {
                    bLedOn = !bLedOn;
                    colorSensor.enableLed(bLedOn);
                    bPrevState = bCurrState;
                    Color.RGBToHSV(colorSensor.red() * 8, colorSensor.green() * 8, colorSensor.blue() * 8, hsvValues);
                }
            } else {

                frontLDrive.setPower(leftFrontPower * 1);
                frontRDrive.setPower(rightFrontPower * 1);
                backLDrive.setPower(leftBackPower * 1);
                backRDrive.setPower(rightBackPower * 1);

                relativeLayout.post(new Runnable() {
                    public void run() {
                        relativeLayout.setBackgroundColor(Color.HSVToColor(0xff, values));
                        relativeLayout.setBackgroundColor(Color.WHITE);
                    }


                });
                telemetry.addData("Status", "Run Time: " + runtime.toString());
                telemetry.addData("Motors", "front L , front R , back L , back R , Arm R , Arm L ", frontLDrive, backLDrive, frontRDrive, backRDrive, dreaptaMDrive, stangaMDrive);
                telemetry.update();
                telemetry.addData("LED", bLedOn ? "On" : "Off");
                telemetry.addData("Clear", colorSensor.alpha());
                telemetry.addData("Red  ", colorSensor.red());
                telemetry.addData("Green", colorSensor.green());
                telemetry.addData("Blue ", colorSensor.blue());
                telemetry.addData("Hue", hsvValues[0]);
                telemetry.update();
            }
        }
    }
}
