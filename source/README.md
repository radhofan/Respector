### NOTE: THIS REPO IS INTENDED AS A REPRODUCIBLE PAPER EXPERIMENT AND IS INTENDED TO BE RAN INSIDE A TERMINAL IN EITHER TROVI/BINDER
### THIS README.MD MAY HAVE MORE INSTRUCTIONS THAN THE ORIGINAL REPO TO ENSURE PERFECT REPLICATION
### REPRODUCE THIS REPO IN TROVI VIA THIS LINK: https://chameleoncloud.org/experiment/share/6cbb4948-da03-44c7-b139-e2d086e26094
### REPRODUCE THIS REPO IN BINDER VIA THIS LINK: https://mybinder.org/ 
    - GitHub Repository: `[Insert GitHub Repo Link Here]`  
    - Notebook File (`.ipynb`):** `[Insert Notebook File Name Here]`  
### ORIGINAL REPO LINK: https://github.com/seal-research/gluetest

# GlueTest: Testing Code Translation via Language Interoperability

# Dependencies

- Install Java 11.
- Install Java 8.
- Install Maven.
- Install Gradle.
- Install Z3 (following pre-built Z3 instruction below)

## Installations

Install Java:
```bash
sudo apt update
sudo apt install -y openjdk-11-jdk openjdk-8-jdk
```

Install Maven:
```bash
sudo apt install -y maven
```

Install Gradle:
```bash
wget https://services.gradle.org/distributions/gradle-8.5-bin.zip -O gradle.zip
sudo unzip gradle.zip -d /opt/gradle
echo 'export PATH=/opt/gradle/gradle-8.5/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Prebuilt Z3:
```bash
wget https://github.com/Z3Prover/z3/releases/download/z3-4.12.2/z3-4.12.2-x64-ubuntu-18.04.zip -O z3.zip
sudo apt install -y zip unzip
unzip z3.zip
sudo mv z3-4.12.2-x64-ubuntu-18.04 /opt/z3
echo 'export PATH=/opt/z3/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

## Pre-built Z3

For your convenience, we provided the pre-built binary of the version of Z3 we used for Respector: [z3.zip](). You can download and decompress `z3.zip`.

You can set the root folder of the extracted files as `Z3_HOME` to other scripts with the following command.

```
export Z3_HOME=<path to the clone repo>
```

This binary of Z3 was built on Ubuntu 20.04 on AMD64 machine with Java 11.

## Download and install Z3

Clone Z3 from https://github.com/Z3Prover/z3

We used the version of Z3 with commit ID `23c53c6820b2d0c786dc416dab9a50473a7bbde3`.

Inside the folder of cloned Z3, execute the following commands to build Z3 and set the environment variable `Z3_HOME` for other scripts.


```
git checkout 23c53c6820b2d0c786dc416dab9a50473a7bbde3
python scripts/mk_make.py --java
cd build
make
export Z3_HOME=<path to the clone repo>
```

# How to compile Respector?

1. Specify the path to the Java binding of `Z3` on line 51 in pom.xml. You are supposed to provide `$Z3_HOME/build/com.microsoft.z3.jar` as the `<systemPath>` to the Maven dependency of `Z3`.

2. You might have installed some version of Soot in your Maven local repo. Try to delete folder `~/.m2/repository/org/soot-oss/`, otherwise it is using the Soot there instead of the version we provided in `./lib/local_repo/`.

3. Compile and package the project by
   ```
   mvn package
   ```

The complied Jar of Respector would be available in target folder. (Pre-compiled Respector exists at `target/Respector-0.1-SNAPSHOT.jar`)

# How to run Respector?

1. Following the instructions in the README file under the `dataset` folder, compile the APIs you want to evaluate on:

   E.g.,

   ```
   cd ../dataset/restcountries/
   mvn package -DskipTests
   ```

2. To run Respector on an API, use script `run_respector.sh` under `scripts` folder:

   ```
   bash ./scripts/run_respector.sh <path to API class files>...  <path to the generated OAS>
   ```

   `<path to API class files>`: You can provide one or more directories containing relevant class files of the API, including class files of the libraries it uses. You can specify multiple paths as different arguments. The last argument is considered as the path to output the generated specification.


3. Alternatively, to run Respector on all the 15 APIs in the dataset:
   
   Execute `run_all.sh` in `scripts` folder.

   ***NOTE: This will take 5.5 hours in total.***

   ```
   bash ./scripts/run_all.sh ./generated/ ../dataset/
   ```

We have attached the generated specifications in `./generated/` folder. 

***If you see your generated specification different from what we attacthed***: Usually that is a different ordering of endpoint methods and global variables, which depends on how Soot loads the class files. It is normal to have such differences when you are using a different Soot version (which should be avoided) or if you have recompiled the target classes. If you see a different order, you can check the total number of lines in the generated specification. If it is the same as the total number of lines in the specification we attached, then it is just the ordering issue. You can use `./scripts/cmp_lines.sh` to compare the numbers of lines of files between two folders:

```
bash ./scripts/cmp_lines.sh ./generated ./generated_0
```

# Example Use

Here we show the steps of running Respector on Senzing API to generate OpenAPI specifications as described in the paper.

First, please compile the target API with its proper Java commonds.

   Under `../dataset/senzing-api-server`:

   ```
   mvn compile
   ```

The compiled class files would be under `../dataset/senzing-api-server/target/classes`

Run Respector on these class files:

   ```
   bash ./scripts/run_respector.sh ../dataset/senzing-api-server/target/classes ./generated/senzing.json
   ```

It will take around 30 minutes to finish. While executing, Respector will print out how many endpoints it has processed:

![](./documentation/screenshot_progress.png)

Once it is done, you can find the generated OAS at `./generated/senzing.json`.

The endpoint method we use in the motivating example in the paper, `GET /entity-networks` can be found on line 1549:

![](./documentation/screenshot_endpoint.png)

Its enhanced OAS can be found on line 4399:

![](./documentation/screenshot_enhanced.png)

`x-global-variables-info` section is from line 6307 on:

![](./documentation/screenshot_global_var.png)

`x-endpoint-interdependence` section starts from line 5679:

![](./documentation/screenshot_interdependence.png)

# Troubleshooting
Please refer to `TROUBLESHOOTING.md`.