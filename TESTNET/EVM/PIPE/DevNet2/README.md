---

# **Pipe POP Node Setup Guide**

Welcome to the **Pipe POP Node Setup**! This guide will walk you through the entire process of setting up your Pipe POP node, including configuring automatic startup using `systemd`, and utilizing the referral and reputation systems to earn rewards.

---

## **🖥️ Requirements**

Before we get started, make sure you have the following:

- A **Linux system** (Ubuntu is recommended)
- The **`pop` binary** file that was sent to you via email

---

## **⚙️ Setup Instructions**

### **1. Create the Working Directory**

Let's begin by creating the directory where your `pop` node files will reside, and a separate directory for the cache:

```bash
mkdir -p $HOME/pipenetwork/download_cache
cd $HOME/pipenetwork
```

**🔔 Important:** After creating the `pipenetwork` directory, move the **`pop` binary** that you received via email into `$HOME/pipenetwork` directory.

---

### **2. Set Up the Systemd Service**

Now we’ll configure the systemd service to manage the Pipe POP node automatically.

1. **Create the systemd service file** at `/etc/systemd/system/pipe-pop.service` with the following content:
```
sudo tee /etc/systemd/system/pipe-pop.service > /dev/null << EOF
[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
User=$USER
Group=$USER
WorkingDirectory=$HOME/pipenetwork
ExecStart=$HOME/pipenetwork/pop \
    --ram <your-RAM-in-Gb> \
    --max-disk <your-max-disk-in-Gb> \
    --cache-dir $HOME/pipenetwork/download_cache \
    --pubKey <your-solana-address> \
    --signup-by-referral-route d04873036056df2b
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=dcdn-node

[Install]
WantedBy=multi-user.target
EOF
```

   **🌟 Explanation of the Parameters**:
   - `--ram <your-RAM-in-Gb>`: Replace `<your-RAM-in-Gb>` with the amount of RAM you wish to allocate (e.g., `8` for 8GB).
   - `--max-disk <your-max-disk-in-Gb>`: Replace `<your-max-disk-in-Gb>` with the disk space you want to allocate (e.g., `500` for 500GB).
   - `--pubKey <your-solana-address>`: Replace `<your-solana-address>` with your actual Solana public address.


### **3. Enable and Start the Service**

Once your systemd service file is created, it’s time to enable and start the service to ensure it runs automatically on boot:

1. **Reload systemd** to apply the changes:

    ```bash
    sudo systemctl daemon-reload
    ```

2. **Enable the service** to start on boot:

    ```bash
    sudo systemctl enable pipe-pop
    ```

3. **Start the service**:

    ```bash
    sudo systemctl start pipe-pop
    ```

---

### **4. Verify the Service**

To make sure the service is running smoothly, use the following command to check its status:

```bash
sudo systemctl status pipe-pop
```

---

### **5. Monitoring and Logs**

You can monitor the logs and the real-time status of your node by running:

```bash
journalctl -u pipe-pop -f
```

---

### **6. Check Your Node's Reputation and Points**

To check your node's performance, reputation, and points earned from referrals, use the following command:

```
$HOME/pipenetwork/pop --status
```

```
$HOME/pipenetwork/pop --points-route
```

You’ll get a detailed breakdown of your node’s reputation metrics, including:

- **Uptime Score** (40% of total)
- **Egress Score** (30% of total)
- **Historical Score** (30% of total)

---

### **7. Generate Your Referral Code**

Want to share your referral code and earn points? Use the following command to generate your unique referral code:

```bash
./pop --gen-referral-route
```

Once generated, you can share this referral code with others who can sign up and join your network!

---

## **🎉 Congratulations!**

You've successfully set up your **Pipe POP Node** and are now ready to start earning rewards through the referral and reputation systems.

If you have any issues or need further assistance, feel free to reach out to our support team.

---
