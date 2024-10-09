# Software Installation

## 1. System Requirements

To run the *dnbc4tools* package on a Linux-based system, the following minimum requirements are necessary:

- **x86-64 compatible processor**
- **50 GB RAM** and **4 CPUs**
- **CentOS 7.x** 64-bit operating system (Linux kernel 3.10.0) or a compatible newer version

</br>
</br>

## 2. Software Download

### dnbc4tools 2.1.3 (Released: October 9, 2024)

**Download for Linux 64-bit**: [dnbc4tools2.1.3.tar.gz](https://ftp.cngb.org/pub/CNSA/data5/CNP0006367/Single_Cell/CSE0000448/dnbc4tools2.1.3.tar.gz)

- **File size**: 495M
- **md5sum**: dfda9a3f308aaa3fdaa6c2a971bb2821

```shell
#### Download using `wget`
wget -O dnbc4tools2.1.3.tar.gz "https://ftp.cngb.org/pub/CNSA/data5/CNP0006367/Single_Cell/CSE0000448/dnbc4tools2.1.3.tar.gz"
#### Download using `curl`
curl -o dnbc4tools2.1.3.tar.gz "https://ftp.cngb.org/pub/CNSA/data5/CNP0006367/Single_Cell/CSE0000448/dnbc4tools2.1.3.tar.gz"
```

**New Features in dnbc4tools 2.1.3**

- Available as a tar.gz compressed file, requiring no additional configuration.
- **New Modules**: Single-cell RNA 5' Transcriptome Analysis and Single-cell VDJ Analysis.

</br>
</br>

## 3. Software Installation

*dnbc4tools* is distributed as a tar.gz package that can be directly extracted and run without any additional setup. All necessary dependencies are precompiled, making the software compatible with most Linux environments.

1. **Download and extract the dnbc4tools package** to a suitable directory. In this example, it will be extracted to `/opt/software`.

   ```shell
   cd /opt/software
   # Extract the dnbc4tools package
   tar -xzvf dnbc4tools2.1.3.tar.gz
   ```

2. During extraction, numerous files will be listed in the terminal. Once extraction is complete, the directory should contain the following items, with `dnbc4tools2.1.3/dnbc4tools` being the executable:

   ```shell
   dnbc4tools2.1.3/
   dnbc4tools2.1.3/dnbc4tools
   dnbc4tools2.1.3/external
   dnbc4tools2.1.3/lib
   dnbc4tools2.1.3/misc
   dnbc4tools2.1.3/sourceC4.bash
   ```

This completes the installation. *dnbc4tools* is now ready for use.