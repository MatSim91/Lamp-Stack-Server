<h1 align="center">LAMP Stack Server</h1>

**Overview:** The goal of this project is to configure and run an Apache HTTP server in a Linux OS Server, set up Git for version control, create a Relational Database, run containerized Node.js applications with Docker, use Kubernetes as container orchestration, and create CI/CD pipelines and Python automation.

I will be developing from scratch a customized LAMP stack first locally and then eventually connecting the entire application to the internet and will be documenting all my learning paths with the steps, errors, and cool solutions that I have found.

# Table Of Contents

1. [Finding a Machine](#finding-a-machine)
2. [Linux Installation](#linux-installation)
3. [Apache Installation](#apache-installation)
4. [Additional Tools and Packages](#additional-tools-and-packages)
    - [Hardware Lister](#hardware-lister)
5. [Technologies Used](#technologies-used)
6.  [Languages Used](#languages-used)
7.  [Frameworks, Libraries & Programs Used](#frameworks-libraries-and-programs-used)
8.  [Testing](#testing)
    - [](#)
9. [Deploy](#deploy)
10. [Credits](#credits)
    - [](#)


# Finding a Machine

My first thought regarding which machine to use to start the server was about buying a Rasperry Pi, but due to the high price and no stock availability on the sites I have searched I decided not to get one. Then I remembered about an old HP Probook Laptop that was sitting in the closet gathering dust, and the configuration was not bad at all to start this project:

```
HP ProBook 4540 
Intel(R) Core(TM) i3-3110M CPU @ 2.40GHz
4GiB SODIMM DDR3 Memory RAM
500GB Hitachi HTS72755
16GB Cruzer Fit
```

My recommendation: With the chip shortage and the high demand for Raspberry Pi's the prices are crazy. Instead you can use that old abandoned PC gathering dust for free!

# Linux Installation

# Apache Installation

# Additional Tools and Packages

  ## Hardware Lister
  
  It is a tool that provides detailed information on the hardware configuration of the machine.
  
  Site and Documentation: https://ezix.org/project/wiki/HardwareLiSter
  
  **To install:**

  `sudo apt-get install lshw`
  
  **To check the hardware information:**

  `sudo lshw -short`
