# DNS-Server-Setup-Implementation

<br>


## Prerequisites for DNS Server Setup (CentOS 9)

1. Linux Server VM:
      - OS: CentOS 9 with static IP configured.
      - Role: DNS server running BIND.

2. Linux Client VM:

      - OS: CentOS 9.
      - Role: DNS client for testing resolution.
      - Network Configuration: Configure the client to use the server VM's IP as its DNS resolver.




<br>







## Introduction to DNS

*Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the internet or a private network. DNS translates human-readable domain names (e.g., example.com) into IP addresses that computers use to identify each other.*

## Key Components:

- **Forward Zone:** Maps domain names to IP addresses.

- **Reverse Zone:** Maps IP addresses to domain names.

- **BIND:** Software that provides an implementation of DNS services.






<br>


# *******************  Implementation Steps  **********************


<br>

---

## Step 1: Network Configuration

1. Open network configuration:

   ```yml
   nmtui
   ```

   Use the `nmtui` command to configure IP, Gateway, DNS, and hostname interactively.

2. Set the following details:

   - **IP Address:** Assign a static IP to the server.
   - **Gateway:** Use `route -n` to determine the gateway.
   - **Primary DNS:** Server’s own IP address.
   - **Secondary DNS:** `8.8.8.8` (Google's public DNS).
   - **DNS Search Domain:** Specify a domain name (e.g., `suraj.com`).
   - **Hostname:** Set a unique machine name.

3. Restart network services:

   ```yml
   systemctl restart network       # OR
   systemctl restart NetworkManager
   ```

   These commands ensure the updated network configuration is applied.

4. Enable the network interface:

   ```yml
   ip link set ens160 up
   ```

   Replace `ens160` with your network interface name.

---

## Step 2: Install BIND

Install BIND (Berkeley Internet Name Domain), the software for DNS services:

```yml
sudo dnf install bind bind-utils -y
```

- `bind-utils`: Includes useful DNS query utilities.
- `-y`: Automatically confirms the installation.

---

## Step 3: Configure BIND

Edit the primary configuration file:

```yml
vim /etc/named.conf
```

Modify the configuration:

```yml
options {
    listen-on port 53 { 192.168.82.212; };                          # In my case  Server IP is 192.168.82.212
    directory       "/var/named";
    allow-query     { any; };
    recursion yes;
    dnssec-validation yes;
    managed-keys-directory "/var/named/dynamic";
};

zone "suraj.com" IN {
    type master;
    file "for.suraj.com";
};

zone "82.168.192.in-addr.arpa" IN {
    type master;
    file "rev.suraj.com";
};
```

- `listen-on port 53`: Binds the DNS service to the specified IP.
- `allow-query { any; }`: Allows DNS queries from all clients.
- `zone`: Defines forward and reverse zones for DNS resolution.

Restart the service:

```yml
systemctl restart named
```

---

## Step 4: Create Zone Files

Navigate to the DNS directory:

```yml
cd /var/named
```

### Forward Zone File

1. Copy the default file:
   
   ```yml
   cp -av named.localhost for.suraj.com
   ```
3. Edit the forward zone file:
   
   ```yml
   vim for.suraj.com
   ```
   Content:

   ```yml
   $TTL 1D
   @       IN SOA   suraj.com.        root.suraj.com. (
                            0           ; serial
                            1D          ; refresh
                            1H          ; retry
                            1W          ; expire
                            3H )        ; minimum

           IN NS    master.suraj.com.

   suraj.com.        IN A     192.168.82.212
   master.suraj.com. IN A     192.168.82.212
   ```

### Reverse Zone File

1. Copy the default file:
   
   ```yml
   cp -av named.localhost rev.suraj.com
   ```

3. Edit the reverse zone file:
   
   ```yml
   vim rev.suraj.com
   ```
   Content:

   ```yml
   $TTL 1D
   @       IN SOA   suraj.com.        root.suraj.com. (
                            0           ; serial
                            1D          ; refresh
                            1H          ; retry
                            1W          ; expire
                            3H )        ; minimum

           IN NS    master.suraj.com.

   212     IN PTR   suraj.com.
   212     IN PTR   master.suraj.com.
   ```

---

## Step 5: Restart and Verify

1. Restart the DNS service:
   
   ```yml
   systemctl restart named
   ```

3. Verify DNS resolution:
   
   ```yml
   host suraj.com
   host master.suraj.com
   ```

---

## Troubleshooting

1. If hostname resolution fails, check the resolver configuration:
   
   ```yml
   cat /etc/resolv.conf
   ```
3. Ensure the file contains:
   
   ```yml
 
   search suraj.com master.suraj.com
   nameserver 192.168.82.212
   nameserver 8.8.8.8
   ```


<br>
<br>

# ******************* Screenshots Implementation **********************



## 1.
<br>
<br>


![1 nmtui](https://github.com/user-attachments/assets/c3d8843d-815e-4e1e-a711-fcdd5789c23b)







<br>
<br>

 ## 2. 


![2 2 resolv conf ](https://github.com/user-attachments/assets/edec5f83-9291-427d-ba3d-c3ef18057cdd)




<br>
<br>

## 3.






![2   named conf](https://github.com/user-attachments/assets/9038ab91-96c6-4f1c-bd7f-cbc7a7d7c0a8)






<br>
<br>

## 4.
  

![3 1  forward file ](https://github.com/user-attachments/assets/d1b7219e-aea9-4c44-826a-aacfb3ef5e07)





<br>
<br>

## 5.




![3 2  reverse file ](https://github.com/user-attachments/assets/ff571186-4923-4291-839b-4e21a7b89a23)








<br>
<br>


  ## 6.


![ip and resolve host](https://github.com/user-attachments/assets/0f9dd2b4-332e-4e86-8ba4-b9cb6edeffe1)




<br>
<br>

## 7.




![final verification on server](https://github.com/user-attachments/assets/a96293d5-a08c-4e4d-8d90-578b6df32085)








<br>
<br>
  
<br>



# ************************ Client Setup ******************************




<br>




## Step 1: Network Configuration

1. Open network configuration:

   ```yml
   nmtui
   ```

   Use the `nmtui` command to configure IP, Gateway, DNS, and hostname interactively.

2. Set the following details:

   - **IP Address:** Assign a static IP to the server.                                 ```yml  # In my case  192.168.82.145  ```
   - **Gateway:** Use `route -n` to determine the gateway.
   - **Primary DNS:** Server’s DNS IP     # In my case  192.168.82.212
   - **Secondary DNS:** `8.8.8.8` (Google's public DNS).
   - **DNS Search Domain:** Specify a domain name (e.g., `suraj.com`).
   - **Hostname:** Set a unique machine name.

3. Restart network services:

   ```yml
   systemctl restart network       # OR
   
   systemctl restart NetworkManager
   ```

   These commands ensure the updated network configuration is applied.

4. Enable the network interface:

   ```yml
   ip link set ens160 up
   ```

   Replace `ens160` with your network interface name.

---


<br>
<br>
<br>
<br>

# ******************* Screenshots Implementation **********************

<br>


## 1. 

<br>
<br>
  
![i   nmtui](https://github.com/user-attachments/assets/7c71e52e-fe86-4169-9d38-bdf895fca4b4)






<br>
<br>

## 2.





![2   ip and route](https://github.com/user-attachments/assets/922ebfe7-4157-4224-9b3c-2bd943ef0876)







<br>
<br>

  ## 3. 



![final verification](https://github.com/user-attachments/assets/be957055-dfc6-4977-ba60-52f4fda0e57c)



<br>
<br>
































