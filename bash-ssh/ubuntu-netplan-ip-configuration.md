# Configuring a Static IP Address with Netplan

Netplan is the network configuration system commonly used by Ubuntu. Its configuration files are written in YAML and stored in:

```bash
/etc/netplan/
```

Typical Netplan files include:

```text
/etc/netplan/01-network-manager-all.yaml
/etc/netplan/50-cloud-init.yaml
```

## 1. Check the Network Interface Name

Before configuring a static IP address, identify the network interface name.

```bash
ip link
```

or:

```bash
ip addr
```

Example interface names:

```text
ens33
ens160
enp0s1
eth0
```

In this example, the interface name will be:

```text
enp0s1
```

---

## 2. Check Existing Netplan Files

List the existing Netplan configuration files:

```bash
ls -l /etc/netplan/
```

Example output:

```text
-rw------- 1 root root 104 Jul 18 20:00 01-network-manager-all.yaml
-rw------- 1 root root 230 Jul 18 20:00 50-cloud-init.yaml
```

Netplan files are normally owned by `root` because they control system-wide network configuration.

You can view them with:

```bash
sudo cat /etc/netplan/01-network-manager-all.yaml
sudo cat /etc/netplan/50-cloud-init.yaml
```

You can edit them with:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

or:

```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```

### File Ownership

It is normally unnecessary to change the file owner. Using `sudo` is the recommended method.

If the file permissions prevent a normal user from reading or editing the file, use:

```bash
sudo cat /etc/netplan/50-cloud-init.yaml
sudo vim /etc/netplan/50-cloud-init.yaml
```

Changing ownership to the current user is technically possible:

```bash
sudo chown "$USER":"$USER" /etc/netplan/50-cloud-init.yaml
```

However, this is generally not recommended because Netplan configuration files are system files and should remain owned by `root`.

After editing, ownership can be restored with:

```bash
sudo chown root:root /etc/netplan/50-cloud-init.yaml
```

The recommended permissions are:

```bash
sudo chmod 600 /etc/netplan/50-cloud-init.yaml
```

---

## 3. Understand Multiple Netplan YAML Files

Netplan reads and merges all YAML files located in:

```text
/etc/netplan/
```

The files are processed in lexicographical filename order.

For example:

```text
01-network-manager-all.yaml
50-cloud-init.yaml
99-static-ip.yaml
```

They are processed in this order:

```text
01 → 50 → 99
```

When multiple files define the same key or network interface setting, a later file may override values from an earlier file.

For example, `50-cloud-init.yaml` is processed after `01-network-manager-all.yaml`.

A common `01-network-manager-all.yaml` file looks like this:

```yaml
network:
  version: 2
  renderer: NetworkManager
```

This file only specifies that NetworkManager should manage the network.

A `50-cloud-init.yaml` file may contain the actual interface configuration:

```yaml
network:
  version: 2
  ethernets:
    enp0s1:
      dhcp4: true
```

To avoid confusing merged settings, it is best to define a specific interface in only one Netplan file.

---

## 4. Configure a Static IP Address

You can modify the existing interface configuration or create a separate file.

For a separate static IP configuration:

```bash
sudo vim /etc/netplan/99-static-ip.yaml
```

Example configuration:

```yaml
network:
  version: 2
  renderer: NetworkManager

  ethernets:
    enp0s1:
      dhcp4: false
      addresses:
        - 192.168.64.10/24

      routes:
        - to: default
          via: 192.168.64.1

      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Make sure to replace the following values with values appropriate for the network:

```text
enp0s1           Network interface
192.168.64.10    Static IP address
/24              Network prefix
192.168.64.1     Default gateway
1.1.1.1          DNS server
8.8.8.8          Secondary DNS server
```

---

## 5. Configuration Explanation

### Disable DHCP

```yaml
dhcp4: false
```

This disables automatic IPv4 address assignment through DHCP.

### Static Address

```yaml
addresses:
  - 192.168.64.10/24
```

This assigns `192.168.64.10` to the interface with a `/24` prefix.

A `/24` prefix is equivalent to:

```text
255.255.255.0
```

### Default Route

```yaml
routes:
  - to: default
    via: 192.168.64.1
```

This sets `192.168.64.1` as the default gateway.

The older `gateway4` option may still be seen in older examples, but the `routes` format is preferred.

### DNS Servers

```yaml
nameservers:
  addresses:
    - 1.1.1.1
    - 8.8.8.8
```

This configures the DNS resolvers used for domain name resolution.

---

## 6. NetworkManager vs. systemd-networkd

Ubuntu Desktop usually uses:

```yaml
renderer: NetworkManager
```

Ubuntu Server commonly uses:

```yaml
renderer: networkd
```

A server-style configuration may look like this:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    enp0s1:
      dhcp4: false
      addresses:
        - 192.168.64.10/24
      routes:
        - to: default
          via: 192.168.64.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Do not change the renderer without a reason. It is usually best to keep the renderer already configured by the system.

---

## 7. Generate the Backend Configuration

Run:

```bash
sudo netplan generate
```

`netplan generate` reads the YAML files under:

```text
/etc/netplan/
```

It validates and converts them into configuration files used by the selected backend:

```text
Netplan YAML
    ↓
netplan generate
    ↓
NetworkManager or systemd-networkd configuration
```

For `systemd-networkd`, generated files are commonly written under:

```text
/run/systemd/network/
```

For NetworkManager, generated runtime configuration is placed under its corresponding `/run` configuration directories.

Running `netplan generate` does not normally activate the new network configuration immediately.

It is useful for detecting problems such as:

- Invalid YAML syntax
    
- Incorrect indentation
    
- Unknown Netplan properties
    
- Invalid address or route formats
    
- Conflicting configuration structures
    

Example:

```bash
sudo netplan generate
```

If the command produces no error, the configuration was successfully generated.

---

## 8. Test the Configuration Safely

Use:

```bash
sudo netplan try
```

This temporarily applies the new network configuration.

Netplan asks you to confirm the configuration within a limited time. If you do not confirm it, Netplan attempts to restore the previous network configuration.

This is especially useful when changing network settings over SSH because an incorrect IP address or gateway could disconnect the session.

Conceptually:

```text
netplan try
= temporarily apply the configuration
= wait for confirmation
= automatically roll back if not confirmed
```

---

## 9. Apply the Configuration

To apply the new configuration permanently:

```bash
sudo netplan apply
```

Conceptually:

```text
netplan generate
= generate backend configuration only

netplan apply
= generate and activate the configuration

netplan try
= temporarily activate with rollback protection
```

A safe workflow is:

```bash
sudo netplan generate
sudo netplan try
sudo netplan apply
```

After successfully confirming `netplan try`, an additional `netplan apply` may not always be necessary, but it can be used to ensure that the current YAML configuration is active.

---

## 10. Verify the Network Configuration

Check the assigned IP address:

```bash
ip addr show enp0s1
```

Check the routing table:

```bash
ip route
```

Expected route output:

```text
default via 192.168.64.1 dev enp0s1
192.168.64.0/24 dev enp0s1 proto kernel scope link src 192.168.64.10
```

Check the complete merged Netplan configuration:

```bash
sudo netplan get
```

Check DNS configuration:

```bash
resolvectl status
```

Test the gateway:

```bash
ping -c 4 192.168.64.1
```

Test Internet connectivity without DNS:

```bash
ping -c 4 1.1.1.1
```

Test DNS resolution:

```bash
ping -c 4 google.com
```

These tests help separate different problems:

```text
Gateway ping fails
→ Local network, IP, subnet, interface, or gateway problem

1.1.1.1 works but google.com fails
→ DNS configuration problem

Both IP and domain tests work
→ Static network configuration is functioning
```

---

## 11. Static IP Without a Gateway

For an additional interface used only for private network practice, the default route and DNS configuration may be omitted.

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    enp0s2:
      dhcp4: false
      addresses:
        - 10.10.10.10/24
```

This interface can communicate with devices in the directly connected `10.10.10.0/24` network, but it does not provide a default route to other networks.

Be careful not to configure multiple default routes unintentionally when using multiple interfaces.

---

## 12. YAML Formatting Rules

Netplan uses YAML syntax.

Use spaces instead of tabs.

Correct:

```yaml
network:
  version: 2
  ethernets:
    enp0s1:
      dhcp4: false
```

Incorrect:

```yaml
network:
	version: 2
```

A common convention is two spaces for each indentation level.

Before applying any configuration, always run:

```bash
sudo netplan generate
```

---

## Recommended Workflow

```bash
# Check the interface name
ip link

# Check existing Netplan files
ls -l /etc/netplan/
sudo cat /etc/netplan/*.yaml

# Edit or create the configuration
sudo vim /etc/netplan/99-static-ip.yaml

# Protect the configuration file
sudo chown root:root /etc/netplan/99-static-ip.yaml
sudo chmod 600 /etc/netplan/99-static-ip.yaml

# Validate the YAML and generate backend configuration
sudo netplan generate

# Temporarily apply with rollback protection
sudo netplan try

# Apply the configuration
sudo netplan apply

# Verify
ip addr
ip route
sudo netplan get
resolvectl status
```

## Key Points

- Netplan configuration files are stored under `/etc/netplan/`.
    
- All YAML files in that directory are merged.
    
- Files are processed in filename order.
    
- A later file can override settings from an earlier file.
    
- Netplan files should normally remain owned by `root`.
    
- Use `sudo` to read or edit root-owned configuration files.
    
- `netplan generate` validates and generates backend configuration.
    
- `netplan apply` activates the configuration.
    
- `netplan try` provides rollback protection.
    
- YAML must use spaces, not tabs.
    
- Avoid defining the same network interface in multiple files unless the merge behavior is intentional.