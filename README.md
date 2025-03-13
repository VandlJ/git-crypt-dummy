# git-crypt-dummy

Dummy repository for testing encrypting files in Git repository.

---

# **🔐 How to Use `git-crypt` for Securely Sharing `.env` Files**  

This guide explains how to set up and use `git-crypt` to encrypt sensitive files in your repository. Each developer must follow these steps to access encrypted files like `.env`.  

## **1️⃣ Installing `git-crypt` and GPG**  
### **💻 macOS (Homebrew)**  
```sh
brew install git-crypt gnupg
```
*(`gnupg` is required for managing GPG keys.)*  

### **🐧 Linux (Debian/Ubuntu)**  
```sh
sudo apt update
sudo apt install git-crypt gnupg
```

### **🖥️ Windows (WSL or Git Bash)**  
#### **Option 1: Using WSL (Recommended)**  
Inside WSL (e.g., Ubuntu):  
```sh
sudo apt update
sudo apt install git-crypt gnupg
```

#### **Option 2: Using Git Bash**  
1. **Download and install GPG:**  
   - [Download GPG for Windows](https://www.gpg4win.org/)  
   - Choose "Full mode" during installation.  
2. **Download and install `git-crypt`:**  
   - Download `git-crypt.exe` from [GitHub releases](https://github.com/AGWA/git-crypt/releases)  
   - Add it to your system `PATH`.  

Then verify the installation:  
```sh
git-crypt --version
gpg --version
```

---

## **2️⃣ Generating a GPG Key**  
Each team member must generate their own **GPG key**.  

```sh
gpg --full-generate-key
```
- **Key type:** RSA and RSA  
- **Key size:** 4096 bits  
- **Expiration:** No expiration  
- **Name and email:** Use your work email  
- **Password:** Choose a secure password  

After generation, list your **Key ID**:  
```sh
gpg --list-secret-keys --keyid-format LONG
```
Look for `sec   rsa4096/ABCDEF1234567890`, where **ABCDEF1234567890** is your Key ID.  

Then, export your **public key** and send it to the repository administrator:  
```sh
gpg --export -a "your-email@example.com" > my-public-key.asc
```
Send **my-public-key.asc** securely via Bitwarden, encrypted email, etc.  

---

## **3️⃣ Adding a New User to `git-crypt` (For Admins)**  
This step is done by the **repository administrator**!  

1️⃣ Import the new user’s public key:  
```sh
gpg --import my-public-key.asc
```

2️⃣ Add the user to `git-crypt`:  
```sh
git-crypt add-gpg-user your-email@example.com
```

3️⃣ Commit and push the changes:  
```sh
git add .git-crypt
git commit -m "Added new user to git-crypt"
git push
```

📌 **Note:** Each new user must be added by an administrator!  

---

## **4️⃣ Unlocking the Repository (`git-crypt unlock`)**  
Once the admin has added you, you can unlock the `.env` file:  

1️⃣ **Clone the repository:**  
```sh
git clone git@github.com:your-repo.git
cd your-repo
```

2️⃣ **Check that you have a GPG key:**  
```sh
gpg --list-secret-keys
```
You should see your key listed.

3️⃣ **Unlock the encrypted files:**  
```sh
git-crypt unlock
```

Now you can see the `.env` file in plain text.

---

## **5️⃣ Encrypting New Files**  
To add more files for encryption, update `.gitattributes`:  

```sh
echo ".env filter=git-crypt diff=git-crypt" >> .gitattributes
```

Then commit the change:  
```sh
git add .gitattributes
git commit -m "Added .env to git-crypt encryption"
git push
```

---

## **6️⃣ What to Do If Someone Loses Access?**  
If someone **loses their GPG key**, an admin can generate an unlock key:  

```sh
git-crypt export-key git-crypt-key
```
Then send it securely. The user can then unlock with:  

```sh
git-crypt unlock git-crypt-key
```

---

## **7️⃣ Removing a User from `git-crypt`**  
If someone should **no longer have access**, you need to reinitialize encryption:  

1. **Decrypt everything and remove `git-crypt`**  
   ```sh
   git-crypt unlock
   git-crypt uninit
   rm -rf .git-crypt
   ```
   
2. **Reinitialize `git-crypt` and add only allowed users**  
   ```sh
   git-crypt init
   git-crypt add-gpg-user new-user@example.com
   ```

3. **Commit and push the changes**  
   ```sh
   git add .git-crypt
   git commit -m "Reset git-crypt keys"
   git push --force
   ```

**Note:** Removing a user does not prevent access to already downloaded files, but it ensures they cannot pull new changes.

---

## **📌 Summary of Key Commands**  
| Action | Command |
|--------|---------|
| Install (macOS) | `brew install git-crypt gnupg` |
| Install (Linux) | `sudo apt install git-crypt gnupg` |
| Generate a GPG key | `gpg --full-generate-key` |
| Export public key | `gpg --export -a "your-email@example.com" > my-public-key.asc` |
| Import a public key | `gpg --import my-public-key.asc` |
| Add a user to `git-crypt` | `git-crypt add-gpg-user your-email@example.com` |
| Unlock the repository | `git-crypt unlock` |
| Add an encrypted file | `echo ".env filter=git-crypt diff=git-crypt" >> .gitattributes` |
| Export an unlock key | `git-crypt export-key git-crypt-key` |
| Reset encryption and remove a user | `git-crypt uninit && rm -rf .git-crypt && git-crypt init` |

---

🔐 **Now your team can securely share the `.env` file without risking sensitive data leaks!**
