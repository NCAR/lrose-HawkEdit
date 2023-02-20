Equations in RadxRay::applyGeorefs (CfRadial), radar_angles from SoloII (), and Wen-Chau's paper Mapping of Airborne Doppler Radar Data.

The equations in section 7.5 of the CfRadial document cover the equations in SoloII radar_angles and are more general than the equations in SoloII radar_angles.  Section 7.5 of the CfRadial document is coded in the RadxRay::applyGeorefs method.

The equations in Wen-Chau's paper are covered by the equations in section 7.5 of the CfRadial document, except for the section 4. Expression of the Doppler velocity.  That section is roughly the code in SoloII::AcVel which calculates the correction for RemoveAircraftMotion.  The other section of Wen-Chau's paper that does not seem to be covered by the RadxRay::applyGeorefs method is the case when "the pitch and heading are changing with time and the INS is located some distance L away from the antenna" (apparent antenna motion).  I didn't find the equations for this correction in either the SoloII code or the Radx code.  

Proposal to integrate the SoloII code for RemoveAircraftMotion with the Radx georeference code: 
1. replace the SoloII radar_angles code with the RadxRay::applyGeorefs code.
2. Use the elevation calculation from RadxRay::applyGeorefs
3. Calculate the tilt_t in RadxRay::applyGeorefs; store the tilt and rotation angle in the Radx::Georefs structure/class. Note: this would overwrite any existing tilt and rotation.

The SoloII function AcVel is:

d = sqrt( (ew_vel)^2 + (ns_vel)^2 ) + cfac_ew_gndspeed_corr
ac_vel = d * sin (tilt) + vert_vel * sin (elev)

The SoloII function AcVel encodes of this equation ...
V_g = i VG sin T + j VG cos T + k WG.    Equation (19) from Wen-Chau's paper
VG  = horizontal ground speed
WG = vertical ground speed
V_g = the motion vector of the aircraft

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
