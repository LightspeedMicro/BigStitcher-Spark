# BigStitcher-Spark
Running compute-intense parts of BigStitcher distributed. For now we support **fusion with affine transformation models** (including translations of course). It should scale very well to large datasets as it tests for each block that is written which images are overlapping. You simply need to specify the `XML` of a BigSticher project and decide which channels, timepoints, etc. to fuse. *Warning: not tested on 2D yet.*

Sharing this early as it might be useful ...

Here is my example config for this [example dataset](https://drive.google.com/file/d/1SXossOV3gOdIWI4RcDn-YjUYS9SMh5mm/view?usp=sharing) for the main class `net.preibisch.bigstitcher.spark.AffineFusion`:

```
-x '~/test/dataset.xml'
-o '~/test/test-spark.n5'
-d '/ch488/s0'
--UINT8
--minIntensity 1
--maxIntensity 254
--channelId 0
```
*Note: here I save it as UINT8 [0..255] and scale all intensities between `1` and `254` to that range (so it is more obvious what happens). If you omit `UINT8`, it'll save as `FLOAT32` and no `minIntensity` and `maxIntensity` are required. `UINT16` [0..65535] is also supported.*

***Importantly: since we have more than one channel, I specified to use channel 0, otherwise the channels are fused together, which is most likely not desired. Same applies if multiple timepoints are present.***

The blocksize defaults to `128x128x128`, and can be changed with `--blockSize 64,64,64` for example.

And for local spark you need JVM paramters (8 cores, 50GB RAM):

```
-Dspark.master=local[8] -Xmx50G
```
Ask your sysadmin for help how to run it on your cluster. `mvn clean package -P fatjar` builds `target/BigStitcher-Spark-0.0.1-SNAPSHOT.jar` for distribution. ***Importantly, if you use HDF5 as input data in a distributed scenario, you need to set a common path for extracting the HDF5 binaries (see solved issue [here](https://github.com/PreibischLab/BigStitcher-Spark/issues/8)), e.g.***
```
--conf spark.executor.extraJavaOptions=-Dnative.libpath.jhdf5=/groups/spruston/home/moharb/libjhdf5.so
```


You can open the N5 in Fiji (`File > Import > N5`) or by using `n5-view` from the n5-utils package (https://github.com/saalfeldlab/n5-utils).

You can create a multiresolution pyramid of this data using https://github.com/saalfeldlab/n5-spark


***Update: now there is support for non-rigid distributed fusion using `net.preibisch.bigstitcher.spark.NonRigidFusionSpark`***
In order to run it one needs to additionally define the corresponding **interest points**, e.g. `-ip beads` that will be used to compute the non-rigid transformation.
