## Navigation Correction Wash Cycle
(from Alex's navigation correction documentation notes)

### The general steps are as follows:
1. Identify 10 minutes of data in clear air during the inbound or outbound transit of the aircraft not in the storm.
2. Do a few processing steps in Solo using the dorade files (converted from raw) including the "remove aircraft motion" function which applies an initial correction.
3. Convert the dorade files to cfradials (using no additional RadxConvert args).
4. Run the Cai et al code package which uses the cfradials to create cfac files.
5. Put your cfac files in an archive directory where you can keep adding/subtracting from the main cfac file as needed.
6. Repeat this process with the same data but this time use the cfac file before step 2 and then continue on through 5.
7. Keep repeating and adding up the cfacs until the code spits out cfac values near zero.
8. Proceed to working with good data with your final cfac files.

Based on this information, it seems clear that we need a workaround to view data that has not yet had corrections because there are editing steps 
that need to take place in HawkEdit. The commands in step 2 include renaming of fields so that they are called what the code expects them to be called 
(only the first time after starting from raw data). 

### Further Comments from Alex
We do keep using the same data and iteratively applying cfacs. 
However, it is unclear if cfac files keep getting applied to already corrected data since we use the raw VEL field when starting over 
from step 2 and removing aircraft motion. 

#### Question
It is also worth noting that the "apply georefs" argument is never used in the original process when converting to cfradial.
Does anyone know if this means that Solo was taking care of the navigation corrections (in regards to georefs) when
``` 
copy VEL to VG 
remove-aircraft-motion in VG 
copy VG to VR" 
```
is run in Solo? 
Perhaps this is where the "dd radar angles" function comes in which Brenda has worked with a lot in the Solo source code. 

Brenda: 
*Yes, Solo was taking care of the navigation corrections using the dd_radar_angles code.  Before "remove-aircraft-motion in VG" runs, dd_radar_angles code runs and the resulting values for tilt, rotation, etc. are used by the code in remove-aircraft-motion.  Insidently, the remove-only-surface, like functions also use the results of dd_radar_angles.  So, we will need to sort through those details as well.*

Unfortunately, looking through the guide raised more questions than answers, 
but ideally these steps should be helpful to keep in mind as we work towards a solution in HawkEdit. 
I have attached the original guide document for reference.
