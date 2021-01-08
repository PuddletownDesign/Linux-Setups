## Setting up a limited account for SSH access

### on your macbook

1. Open System Preferences
2. Users & Groups
3. Create a sharing only account

## ![new-account](new-account.png

Now that the account is created try connecting via SSH to the MF account.

![sharing-account](/Users/Brent/Desktop/How to set up secure SSH access to your personal laptop/sharing-account.png)

Now control click (right click) on the account you just made and select "advanced options".

1. Set your desired login shell i'm going to use -  `/usr/local/bin/zsh`
2. Set the desired home directory. I want to access my normal user directory - `/Users/Brent`



#### Connect to the new account via SSH

```bash
ssh guest@10.0.0.35
# or whatever the account you just made is called and the IP address you used last time
```

## 