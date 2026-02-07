# Step 5 — Installing Docker

Now that my system is fully updated and secured, it is time to install Docker.  
Before beginning, I restarted my VM to ensure all updates and services from Step 4 were applied cleanly.

Docker installation on Ubuntu requires three key tools:

- **ca-certificates** — Allows Ubuntu to validate HTTPS connections  
- **curl** — A command-line tool that downloads files from URLs  
- **gnupg (GPG)** — Used to verify digital signatures and ensure downloaded files are authentic  

When I attempted to install these prerequisites, Ubuntu reported that all three were **already installed and up to date**.  
This confirms that my system is fully capable of handling secure downloads and validating signatures before installing Docker.

![installation ready](/images/step5-hello-world.png)

##  Step 5.1 — Making the Docker Installation Secure

Docker packages must be downloaded from an authenticated and trusted source.  
To accomplish this, I followed Docker’s official installation steps, which add a **verified GPG signing key** and a **trusted repository**.

---

### **1) Create a directory for Docker’s key**

sudo install -m 0755 -d /etc/apt/keyrings

Purpose:
This creates a secure directory where Ubuntu stores trusted software keys.
By setting specific permissions (0755), I ensure that the system’s package manager (apt) can read the key without exposing it to unauthorized changes.
This step is about building a chain of trust before installing anything.

### **2) Download and store Docker’s official GPG key **

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

What this does:

curl downloads Docker’s official key from a secure HTTPS endpoint
Flags used:

-f → fail silently if an error occurs
-s → silent (no progress bar)
-S → show errors when they occur
-L → follow redirects if the URL has changed


The | (pipe) sends the output directly into gpg
gpg --dearmor converts the key from text → binary format
The final key is stored safely in /etc/apt/keyrings/docker.gpg

Why this matters:
APT (Ubuntu’s package manager) only installs software if the signatures match.
This ensures the Docker packages I download later are authentic and not tampered with.

### **3) Allow the system to read the key*
Shellsudo chmod a+r /etc/apt/keyrings/docker.gpgShow more lines
Purpose:
Makes the Docker key readable by all processes on the system (not just root).
APT needs read access so it can verify the Docker repository later.

## Step 5.2 — Adding Docker’s Repository

Now that the signing key is trusted, I need to tell Ubuntu where to download Docker from.

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo prints a new repository entry
$(dpkg --print-architecture) automatically detects my CPU architecture (arm64 on M2 Macs)
signed-by= forces apt to check Docker’s signature using the key I downloaded
$(. /etc/os-release && echo $VERSION_CODENAME) automatically inserts my Ubuntu version name (e.g., noble)
sudo tee writes this configuration into a new file:
/etc/apt/sources.list.d/docker.list

Output is hidden with > /dev/null to keep terminal clean

### At this point, everything was going exactly as Docker documents.

## Step 5.3 — Repository Failure on ARM64 (Critical Issue)

However, the Docker repository for Ubuntu 22.04 (jammy) and 24.04 (noble) returned 404 errors for ARM64 inside a VirtualBox guest.
This means the repository path existed for AMD64 and ARM64 on physical hardware,
but not for ARM64 virtualized under VirtualBox.
This is a known issue in the ARM64 community due to:

- Docker not publishing ARM64 packages consistently
- VirtualBox ARM guests not matching expected architecture identifiers
- Ubuntu 24.04 transitioning repository formats
- ARM64 virtualization still maturing on Apple Silicon
I verified the issue multiple times — the repository simply cannot be used from my environment.

## Step 5.4 — Manual Intervention
Because the automatic creation of docker.list failed in VirtualBox,
I manually created the file:
sudo nano /etc/apt/sources.list.d/docker.list
I tried multiple variations:

- noble with signed-by
- jammy with signed-by
- using [trusted=yes]
- even trying Debian’s ARM64 path

Every attempt resulted the 404 not found error

This confirmed the issue was not my syntax but the repository itself.

## Step 5.5 — The Pivot: Switching to Ubuntu’s Native Docker Package
Since Docker’s upstream repo was unavailable on my system,
I switched to Ubuntu’s official Docker package, which is ARM64 compatible.
I installed it using:
sudo apt update
sudo apt install docker.io -y

This version is maintained by Ubuntu, fully signed, fully secure,
and built for ARM64 across all environments (including virtualized ARM).
This installation worked immediately.

## Step 5.6 — Success! Running the First Container

I tested the instalation with: docker run hello-world
The container downloaded successfully and displayed Docker’s standard greeting message.
This confirms:
- Docker Engine is running
- Docker can pull images
- Networking works correctly
- ARM64 container support is functional

![hello-world](/images/step5-hello-world.png)
