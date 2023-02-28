Equations in RadxRay::applyGeorefs (CfRadial), radar_angles from SoloII (), and Wen-Chau's paper Mapping of Airborne Doppler Radar Data[^1].

The equations in section 7.5 of the  [CfRadial document](https://github.com/NCAR/CfRadial/blob/master/docs/CfRadialDoc.v1.5.20211201.pdf) cover the equations in SoloII radar_angles and are more general than the equations in SoloII radar_angles.  Section 7.5 of the CfRadial document is coded in the RadxRay::applyGeorefs method.

The equations in Wen-Chau's paper are covered by the equations in section 7.5 of the CfRadial document, except for the section 4. Expression of the Doppler velocity.  That section is roughly the code in SoloII::AcVel which calculates the correction for RemoveAircraftMotion.  The other section of Wen-Chau's paper that does not seem to be covered by the RadxRay::applyGeorefs method is the case when "the pitch and heading are changing with time and the INS is located some distance L away from the antenna" (apparent antenna motion).  I didn't find the equations for this correction in either the SoloII code or the Radx code.  

Proposal to integrate the SoloII code for RemoveAircraftMotion with the Radx georeference code: 
1. replace the SoloII radar_angles code with the RadxRay::applyGeorefs code.
2. Use the elevation calculation from RadxRay::applyGeorefs
3. Calculate the tilt_t in RadxRay::applyGeorefs; store the tilt and rotation angle in the Radx::Georefs structure/class. Note: this would overwrite any existing tilt and rotation.

The SoloII function AcVel is:
```
d = sqrt( (ew_vel)^2 + (ns_vel)^2 ) + cfac_ew_gndspeed_corr
ac_vel = d * sin (tilt) + vert_vel * sin (elev)
```
The SoloII function AcVel encodes this equation ...
```
V_g = i VG sin T + j VG cos T + k WG.    Equation (19) from Wen-Chau's paper
VG  = horizontal ground speed
WG = vertical ground speed
V_g = the motion vector of the aircraft
```
Note: I am not sure if this is the appropriate correction for the tail radar. In Wen-Chau's paper, "the track-relative tilt angle \tau_t is important.  The component of the aircraft ground speed V_g in the beam direction is -V_g sin \tau_t.  Knowledge of the \tau_t is required to properly remove the ground velocity component in the Doppler velocity."[^1] Should this equation be used instead of the Equation (19)?

In the SoloII code, tilt = tau_t = asin (y_t),
RadxRay::applyGeorefs does not calculate the tilt.  
Could RadxRay::applyGeorefs calculate the tilt from the xx, yy, zz variables? Yes.
Is y_t = yy? Where y_t = level track-relative coordinate system? Yes.

Tests:
1. start with original Dorade sweep file (swp.X) and an associated cfac file.
2. Use RadxConvert to embed the cfac data as meta data into the output CfRadial file. Set the RadxConvert parameter to apply georeferences.
3. Check the output CfRadial file and verify the cfac information is part of the metadata.  Verify the azimuth, elevation, tilt, and rotation are changed and correct using the equations in the CfRadial v1.5 document, and Wen-Chau's paper, for each ray.
4. Use HawkEdit to open the CfRadial file from step 2. Run the script command REMOVE_AIRCRAFT_MOTION. 
5. Verify the VEL field data have changed according to the AcVel equation in Wen-Chau's paper.
6. Run the script command REMOVE_ONLY_SURFACE. Verify data below the ground surface are set to (missing? or 0?).


### Update from unit test:
Use data from Alex's [Aft data set](https://github.com/Alex-DesRosiers/HawkEdit_Testing_Data/tree/main/DORADES_NC/Aft).
#### 1. Reproduce the results from SoloII.  Using radar_angles, georeference data, and cfac data.  This is unit test real_data_N42RF_TS_reproduce_SoloII_yprime from [RemoveAcMotion_unittest.cc](https://github.com/leavesntwigs/lrose-test/blob/master/libs/Solo/RemoveAcMotion_unittest.cc).  In order to reproduce the field VR from VEL using remove_aircraft_motion, I had to change the remove_aircraft_motion function.  There is some trickery happening here.
a) Nyquist velocity
```
    float eff_unamb_vel = 24.96; // from RADD section of Dorade file; use this value!
    float nyquist_velocity = 0.0; // 48.0437;  NOTE!!! SoloII did NOT have the nyquist velocity!
``` 
this forces the se_remove_ac_motion function to use the eff_unamb_vel for the nyquist.  
  b) The trickery.  The nyquist velocity and the aircraft velocity (ac_vel) are scaled by 100, which provides two decimal points of accuracy in the mod function, to adjust the aircraft velocity by the nyquist velocity. In the original SoloII code, the scale factor is used, instead of a fixed number.  We will have to think about how to set the scale factor, since HawkEdit will use the CfRadial data which is already scaled.
``` 
    scaled_nyqv = nyqv * 100 + 0.5;  // DD_SCALE(nyqv);
    scaled_nyqi = 2*scaled_nyqv;
    scaled_ac_vel = ac_vel * 100 + 0.5; // DD_SCALE(ac_vel);
    adjust = scaled_ac_vel % scaled_nyqi;
    if(abs(adjust) > scaled_nyqv) {
      if (adjust > 0) 
        adjust = adjust - scaled_nyqi;
      else 
        adjust = adjust + scaled_nyqi;
    }
```
This is critical when calculating the amount of aircraft velocity to remove from each value of the field data. 
  c) Follow through with the scaling when adding the adjustment for the aircraft motion:
  ```
          vx = data[ssIdx]*100.0 + adjust;
          if(abs(vx) > scaled_nyqv) {
            if (vx > 0) {
              vx = vx - scaled_nyqi;
	          } else {
              vx = vx + scaled_nyqi;
            }
        }
        newData[ssIdx] = vx/100.0;
 ```
 
#### 2. Resolve the differences between using radar_angles (SoloII) and applyGeorefs (Radx; CfRadial V 1.5 specification).  This is detailed in tests real_data_N42RF_TS_ApplyGeoref_vs_radar_angles_yprime and real_data_N42RF_TS_ApplyGeoref_vs_radar_angles_nonzero_cfacs_yprime from RemoveAcMotion_unittest.cc.

a) The SoloII radar_angles code is missing the matrix M_T "which counterclockwise rotates a coordinate system along the z axis by T"[^1].  The matrix is
```
cosT  sinT  0
-sinT cosT  0
  0    0    1
  ```
  Multiplying the ra_x, ra_y, and ra_z values by the M_T matrix produces values that match the xx, yy, zz values calculated in the RadxRay::applyGeorefs function, for the type Y-Prime radars.  
  b) Once the xx, yy, zz, and corresponding radar_angles variables, ra_x, ra_y, and ra_z * M_T, are calculated, then the tilt, rotation, elevation, and azimuth can be calculated.  The ac_vel calculation in the se_remove_ac_motion function uses tilt and elevation angle.
  
### Next steps
1. Add arguments to HawkEdit REMOVE_AIRCRAFT_MOTION to allow calculations using radar_angles.  This would allow SoloII results to continue.  The default is Radx::applyGeorefs calculation.  Document this SoloII option as a separate workflow. 
 
SoloII workflow: 
```
swp.X ==> RadxConvert; insert cfacs; DO NOT apply georefs ==> CfRadial file
CfRadial file ==> HawkEdit; Edit script; REMOVE_AIRCRAFT_MOTION(..., SoloII = true)
```

Default workflow:
```
swp.X ==> RadxConvert; insert cfacs; apply georefs ==> CfRadial file
CfRadial file ==> HawkEdit; Edit script; REMOVE_AIRCRAFT_MOTION(...)
```
2. Discard SoloII and the radar_angles. No ability to reproduce the SoloII results in HawkEdit.
3. Something else?
 
[^1]: "Mapping of Airborne Doppler Radar Data", by Wen-Chau Lee, Peter Dodge, Frank D, Marks, Jr, Peter H. Hildebrand, 19 November 1992 and 9 August 1993. Journal of Atmospheric and Oceanic Technology, Volume 11, p. 572-578.
