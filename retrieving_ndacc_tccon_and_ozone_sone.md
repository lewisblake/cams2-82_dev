# CAMS2_82: NDACC TCCON and ozone sonde
From Bavo:

Below is an example in bash to get the time series for all sites/instruments for a single file: 

```bash
metadata=$(curl -s -H "Authorization: Bearer $BIRABEARER" https://api.aeronomie.be/cams_opendap/ndacc/ch4/ftir/ftir_ch4_i1dg.nc4.ncoattrjson)

for site in $(jq  '.groups | keys [] '<<< $metadata | tr -d \");do 
	instrument=$(jq ".groups.${site}.groups | keys []" <<< $metadata | tr -d \"); 
	echo Requesting $site.$instrument;
	data=$(curl -s --compressed -H "Authorization: Bearer $BIRABEARER" https://api.aeronomie.be/cams_opendap/ndacc/ch4/ftir/ftir_ch4_i1dg.nc4.json?${site}.${instrument}.GB_measurement_tropo_col_on_GB,${site}.${instrument}.MODEL_measurement_smooth_with_GB_avk_tropo_col_on_GB,${site}.${instrument}.GB_time);
        echo Downloaded data for $site.$instrument: $(jq -c ".$site.$instrument | keys []" <<<$data)
done

```

With some minor adaptations (in orange: file names, variables names download or the depth of the metadata (some files provide data in one group (contestants eg)), this script is applicable to all shared files. 

01.02.24 He said:

The TCCON EGG4 rerun data is now also available: the files are accessible through

/tccon/ch4/ftir/ftir_ch4_i1dg.nc4

/tccon/ch4/ftir/ftir_ch4_i1xh.nc4

/tccon/co2/ftir/ftir_co2_i1dg.nc4

/tccon/co2/ftir/ftir_co2_i1xh.nc4

For TCCON the variable names are different (no averaging kernel applied)
```bash
path=tccon/co2/ftir/ftir_co2_i1dg.nc4

metadata=$(curl -s -H "Authorization: Bearer $BIRABEARER" https://api.aeronomie.be/cams_opendap/${path}.ncoattrjson)

for site in $(jq  '.groups | keys [] '<<< $metadata | tr -d \");do 
	instrument=$(jq ".groups.${site}.groups | keys []" <<< $metadata | tr -d \"); 
	echo Requesting $site.$instrument;
	data=$(curl -s --compressed -H "Authorization: Bearer $BIRABEARER" https://api.aeronomie.be/cams_opendap/${path}.json?${site}.${instrument}.GB_column,${site}.${instrument}.MODEL_measurement_on_GB,${site}.${instrument}.GB_time);
        echo Downloaded data for $site.$instrument: $(jq -c ".$site.$instrument | keys []" <<<$data) NPT=$(jq -c ".$site.$instrument.GB_time | length" <<<$data)
        break
done
```
