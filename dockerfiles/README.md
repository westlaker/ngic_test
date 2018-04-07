ARM64 ONLY 
==========

Steps to build from source (NO pre-built docker images):

Starting in the `ngic_test` directory, run the following command to clone the NGIC code at a specific commit and also pull the required DPDK submodule:

`git submodule update --init --recursive`

Copy the Dockerfiles from demo repo to ngic directory:

`cp dockerfiles/Docker* ngic_arm64/`

To move away from hardcoded IP address configs and test DNS apply the patch
to ngic_arm64, by  useing  -DNGIC_TEST CFLAGS switch to enable, already set in
cp and dp makefiles.


Also for this demo, we are disabling the SDN Controller support.
Make sure it is not enabled.

`sed -i 's/CFLAGS/#CFLAGS/g' ngic_arm64/config/ng-core_cfg.mk`

Now you can use the docker-compose to build the container images:

`docker-compose build`


Once all of the images have built, go back and follow the instructions on the main readme starting at [Starting the Data Plane](https://github.com/adivjoseph/ngic_test#starting-the-data-plane)
