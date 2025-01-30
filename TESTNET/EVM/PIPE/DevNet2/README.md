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
mkdir -p /root/pipenetwork
mkdir -p /root/pipenetwork/download_cache
cd /root/pipenetwork
```

**🔔 Important:** After creating the `pipenetwork` directory, move the **`pop` binary** that you received via email into this directory.

---

### **2. Set Up the Systemd Service**

Now we’ll configure the systemd service to manage the Pipe POP node automatically.

1. **Create the systemd service file** at `/etc/systemd/system/pipe-pop.service` with the following content:

    ```ini
    [Unit]
    Description=Pipe POP Node Service
    After=network.target
    Wants=network-online.target

    [Service]
    User=root
    Group=root
    WorkingDirectory=/root/pipenetwork
    ExecStart=/root/pipenetwork/pop \
        --ram <Your RAM size> \
        --max-disk <Your server disk max> \
        --cache-dir /root/pipenetwork/download_cache \
        --pubKey <Your Solana Address> \
        --signup-by-referral-route 65dcec6a1eb60cf9
    Restart=always
    RestartSec=5
    LimitNOFILE=65536
    LimitNPROC=4096
    StandardOutput=journal
    StandardError=journal
    SyslogIdentifier=dcdn-node

    [Install]
    WantedBy=multi-user.target
    ```

   **🌟 Explanation of the Parameters**:
   - `--ram <Your RAM size>`: Replace `<Your RAM size>` with the amount of RAM you wish to allocate (e.g., `8` for 8GB).
   - `--max-disk <Your server disk max>`: Replace `<Your server disk max>` with the disk space you want to allocate (e.g., `500` for 500GB).
   - `--pubKey <Your Solana Address>`: Replace `<Your Solana Address>` with your actual Solana public address.

---

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

```bash
./pop --status
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
