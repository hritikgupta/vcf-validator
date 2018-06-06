# vcf-validator [![Build Status](https://travis-ci.org/EBIvariation/vcf-validator.svg?branch=develop)](https://travis-ci.org/EBIvariation/vcf-validator)

Validator for the Variant Call Format (VCF) implemented using C++11.

It includes all the checks from the vcftools suite, and some more that involve lexical, syntactic and semantic analysis of the VCF input. If any inconsistencies are found, they are classified in one of the following categories:

* Errors: Violations of the VCF specification
* Warnings: An indication that something weird happened (commas were used instead of colons to split ids) or a recommendation is not followed (missing meta-data)

Please read the wiki for more details about checks already implemented.

## Download

We recommend using the [latest release](https://github.com/EBIvariation/vcf-validator/releases) for the most stable experience using vcf-validator. Along with the release notes, you will find the executables `vcf_validator` and `vcf_debugulator`, which will allow you to validate and fix VCF files.


## Run

### Validator

vcf-validator accepts both compressed and non-compressed input VCF files. Supported compression formats are .bz2, .gz and .z. For other formats such as .zip, the `zcat` command and a pipe can be used (see below).

Reading uncompressed files:
  * File path as argument: `vcf_validator -i /path/to/file.vcf`
  * Standard input: `vcf_validator < /path/to/file.vcf`
  * Standard input from pipe: `cat /path/to/file.vcf | vcf_validator`

Reading compressed files:
  * File path as argument: `vcf_validator -i /path/to/compressed_file.vcf.gz`
  * Standard input: `vcf_validator < /path/to/compressed_file.vcf.z`
  * Standard input from pipe: `zcat /path/to/compressed_file.vcf.zip | vcf_validator`

The validation level can be configured using `-l` / `--level`. This parameter is optional and accepts 3 values:

* error: Display only syntax errors
* warning: Display both syntax and semantic, both errors and warnings (default)
* stop: Stop after the first syntax error is found

Different types of validation reports can be written with the `-r` / `--report` option. Several ones may be specified in the same execution, using commas to separate each type (without spaces, e.g.: `-r summary,database,text`).

* summary: Write a human-readable summary report to a file. This includes one line for each type of error and the number of occurrences, along with the first line that shows that type of error (default)
* text: Write a human-readable report to a file, with one description line for each VCF line that has an error.
* database: Write structured report to a database file. The database engine used is SQLite3, so the results can be inspected manually, but they are intended to be consumed by other applications.

Each report is written into its own file and it is named after the input file, followed by a timestamp. The default output directory is the same as the input file's if provided using `-i`, or the current directory if using the standard input; it can be changed with the `-o` / `--outdir` option.

### Debugulator

There are some simple errors that can be automatically fixed. The most common error is the presence of duplicate variants. The needed parameters are the original VCF and the report generated by a previous run of the vcf_validator with the option `-r database`.

The fixed VCF will be written into the standard output, which you can redirect to a file, or use the `-o` / `--output` option and specify the desired file name.

The logs about what the debugulator is doing will be written into the error output. The logs may be redirected to a log file using `2>debugulator_log.txt` or completely discarded with `2>/dev/null`.

### Examples

Simple example: `vcf_validator -i /path/to/file.vcf`

Full example: `vcf_validator -i /path/to/file.vcf -l stop -r database,stdout -o /path/to/output/folder/`

Debugulator example:
```
vcf_validator -i /path/to/file.vcf -r database -o /path/to/write/report/
vcf_debugulator -i /path/to/file.vcf -e /path/to/write/report/vcf.errors.timestamp.db -o /path/to/fixed.vcf 2>debugulator_log.txt
```




## Build
We support building in 3 platforms: linux, mac, docker (inside linux without installing dependencies)
### Docker (Static Build)

The easiest way to build vcf-validator is using the Docker image provided with the source code. This will create an executable that can be run in any Linux machine.

1. Install and configure Docker following [their tutorial](https://docs.docker.com/engine/getstarted/).
2. Create the Docker image:
    1. Clone this Git repository: `git clone https://github.com/EBIvariation/vcf-validator.git`
    2. Move to the folder the code was downloaded to: `cd vcf-validator`
    3. Build the image: `docker build -t ebivariation/vcf-validator docker/`. Please replace `ebivariation` with your user account if you plan to push this image to [Docker Hub](https://hub.docker.com).
3. Build the executable running `docker run -v ${PWD}:/tmp ebivariation/vcf-validator`. Again, replace `ebivariation` with your user name if necessary.

Executables will be created in the `build/bin` subfolder.

### Linux

In Linux You can obitain staticlly linked binaries of vcf-validator.

#### Dependencies

##### Boost

The dependencies are the Boost library core, and its submodules: Boost.filesystem, Boost.iostreams, Boost.program_options, Boost.regex, Boost.log and Boost.system.
If you are using Ubuntu, the required packages' names will be `libboost-dev`, `libboost-filesystem-dev`, `libboost-iostreams`, `libboost-program-options-dev`, `libboost-regex-dev` and `libboost-log-dev`.

##### ODB

You will need to download the ODB compiler, the ODB common runtime library, and the SQLite database runtime library from [this page](http://codesynthesis.com/products/odb/download.xhtml).

ODB requires SQLite3 to be installed. If you are using Ubuntu, the required packages' names will be `libsqlite3-0` and `libsqlite3-dev`.

To install the ODB compiler, the easiest way is to download the `.deb` or `.rpm` packages, in order to be installed automatically with `dpkg`. Both the ODB runtime and SQLite database runtime libraries can be installed manually running `./configure && make && sudo make install`. This will install the libraries in `/usr/local/lib`.

If you don't have root permissions, please run `./configure --prefix=/path/to/odb/libraries/folder` to specify which folder to install ODB in, then `make && make install`, without `sudo`. Also you will have to provide the path to odb while configuring libodb-sqlite using `./configure --with-libodb=/path/to/odb/libraries`

##### bzip2 and zlib
You will require libbz2.a and libz.a.
For Ubuntu Users, the required padkages' names will be `libbz2-dev` and `zlib1g-dev`.

**Note** : You can easily install dependencies using install_dependencies.sh script. Run it as: `./install_dependencies.sh linux`. You may run `./install_dependencies.sh --help` for help.

#### Compile

The build has been tested on the following compilers:
* Clang 3.9 to 5.0
* GCC 4.8 to 6.0

In order to create the build scripts, please run `cmake` with your preferred generator. For instance, `cmake -G "Unix Makefiles"` will create Makefiles, and to build the binaries, you will need to run `make`. If the ODB libraries were not found during the build, please run `sudo updatedb && sudo ldconfig`.

**Note** : We have removed `BUILD_STATIC` flag from `cmake`.
Cmake will automatically generate build scripts for static build. Also, if ODB has been installed in a non-default location, the option `-DEXT_LIB_PATH=/path/to/external/libraries/folder` must be also provided to the `cmake` command.

Binaries will be created in the `bin` subfolder.

### Mac OSX

On Mac OSX you can obitain a statically linked binaries with only system-libraries linked dynamically.

#### Dependencies

##### Boost

The dependencies are the Boost library core, and its submodules: Boost.filesystem, Boost.iostreams, Boost.program_options, Boost.regex, Boost.log and Boost.system. Use `brew` to install boost using `brew install boost`.

##### ODB
You will need to download the ODB compiler, the ODB common runtime library, and the SQLite database runtime library from [this page](http://codesynthesis.com/products/odb/download.xhtml).

ODB requires SQLite3 to be installed. which can be installed. Which can be installed using `brew install sqlite3`

Pre compiled binaries of ODB compiler for mac osx are provided. Both the ODB runtime and SQLite database runtime libraries can be installed manually running `./configure && make && make install`. This will install the libraries in `/usr/local/lib`.

You may install ODB in some other location, please run `./configure --prefix=/path/to/odb/libraries/folder` to specify which folder to install ODB in, then `make && make install`. Also you will have to provide the path to odb while configuring libodb-sqlite using `./configure --with-libodb=/path/to/odb/libraries`

##### bzip2 and zlib
You will require libbz2.a and libz.a.
Pre-compiled static libraries for mac are not available. You will require to build these from source to obitain static library files. You may simply use `install_dependencies.sh` to make these dependencies.

**Note** : You can easily install dependencies using install_dependencies.sh script. Run it as: `./install_dependencies.sh osx`. You may run `./install_dependencies.sh --help` for help.

#### Compile

In order to create the build scripts, please run `cmake` with your preferred generator. For instance, `cmake -G "Unix Makefiles"` will create Makefiles, and to build the binaries, you will need to run `make`.

If ODB has been installed in a non-default location, the option `-DEXT_LIB_PATH=/path/to/external/libraries/folder` must be also provided to the `cmake` command.
Also you may not get precompiled static libraries libbz2.a and libz.a so you will have to build them from source or better to use `install_dependencies.sh osx` to obitain these libraries.

**Note** - keep all the external libraries `libodb.a`, `libodb-sqlite.a`, `libbz2.a` and `libz.a` in same directory.In case the libraries are installed in non-default location, you will have to pass it to cmake using `-DEXT_LIB_PATH=/absolute/path/to/external/libraries`

Binaries will be created in the `bin` subfolder.


## Deliverables

The following binaries are be created after successful build:

* `vcf_validator`: validation tool
* `vcf_debugulator`: automatic fixing tool
* `test_validator` and derivatives: testing correct behaviour of the tools listed above


## Tests

Unit tests can be run using the binary `bin/test_validator` or, if the generator supports it, a command like `make test`. The first option may provide a more detailed output in case of test failure.

**Note**: Tests that require input files will only work when executed with `make test` or running the binary from the project root folder (not the `bin` subfolder).

## Generate code from descriptors

Code generated from descriptors shall be always up-to-date in the GitHub repository. If changes to the source descriptors were necessary, please generate the Ragel machines C code from `.ragel` files using:

```
ragel -G2 src/vcf/vcf_v41.ragel -o inc/vcf/validator_detail_v41.hpp
ragel -G2 src/vcf/vcf_v42.ragel -o inc/vcf/validator_detail_v42.hpp
ragel -G2 src/vcf/vcf_v43.ragel -o inc/vcf/validator_detail_v43.hpp
```

And the full ODB-based code from the classes definitions using:

```
odb --include-prefix vcf --std c++11 -d sqlite --generate-query --generate-schema --hxx-suffix .hpp --ixx-suffix .ipp --cxx-suffix .cpp --output-dir inc/vcf/ inc/vcf/error.hpp
mv inc/vcf/error-odb.cpp src/vcf/error-odb.cpp
```
