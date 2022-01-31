### How to create subset of archive file

RadxConvert -sweep 1 -field VEL_F -f cfrad.20150626_000730.199_to_20150626_001312.085_SPOL_PunSur_SUR.nc -outdir sweep1_VEL_F


### Using parameter file
set_max_range = TRUE;
max_range_km = 100;

RadxConvert -sweep 1 -field VEL_F -f  cfrad.20150626_000730.199_to_20150626_001312.085_SPOL_PunSur_SUR.nc -params RadxConvert_subset_data.params -outdir sweep1_VEL_F_max_range_100

