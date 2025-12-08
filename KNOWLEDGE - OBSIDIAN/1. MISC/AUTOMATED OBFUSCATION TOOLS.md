
# **LINUX**

## *BASHFUSCATOR*

#### INSTALL

```bash
git clone https://github.com/Bashfuscator/Bashfuscator;
cd Bashfuscator;
pip3 install setuptools==65;
python3 setup.py install --user
```
#### RUN

```bash
bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

# **WINDOWS**
## *DOSfuscation*

#### INSTALL

```powershell-session
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git;
cd Invoke-DOSfuscation;
Import-Module .\Invoke-DOSfuscation.psd1;
Invoke-DOSfuscation;
```
#### RUN

```powershell-session
SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
```

```powershell-session
encoding
```

```powershell-session
1
```


