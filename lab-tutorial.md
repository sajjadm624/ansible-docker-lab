# Lightweight Ansible Lab on macOS (8GB RAM)

**Goal:**  
Create 2 minimal “server” containers to practice Ansible. Full SSH access, Python installed, ready for playbooks.

Alpine is tiny (~5 MB) and uses minimal RAM (~50 MB/container). We’ll create 2 nodes.

---

## Create Docker Nodes

### Node 1
```bash
docker run -d --name ans-node-1 -p 2221:22 --memory="256m" --cpus="0.5" alpine:latest sleep infinity
```

### Node 2
```bash
docker run -d --name ans-node-2 -p 2222:22 --memory="256m" --cpus="0.5" alpine:latest sleep infinity
```

**Explanation:**
- `-p 2221:22` → Expose SSH port to Mac  
- `--memory="256m"` → Limits RAM per container  
- `sleep infinity` → Keeps container running but idle  

Check containers:
```bash
docker ps
```

Expected output shows ports mapped:
```
0.0.0.0:2221->22/tcp ...
0.0.0.0:2222->22/tcp ...
```

---

## Install SSH & Python in Containers

### Enter Node 1
```bash
docker exec -it ans-node-1 sh
```

Inside container:
```bash
apk update
apk add openssh python3 sudo bash
```

Set root password:
```bash
passwd   # enter: ansible1
```

Configure SSH:
```bash
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
```

Create runtime dir for sshd:
```bash
mkdir -p /var/run/sshd
```

Generate host keys:
```bash
ssh-keygen -A
```

Start SSH daemon:
```bash
/usr/sbin/sshd
```

Exit container:
```bash
exit
```

Repeat for Node 2 (`ans-node-2`) and note the password: `ansible2`.

---

## Test SSH Access

From your Mac:
```bash
ssh root@localhost -p 2221   # Node 1
ssh root@localhost -p 2222   # Node 2
```

If you get in, your lab is fully accessible.

Tip: If SSH fails, check inside container:
```bash
ps aux | grep sshd   # Make sure sshd is running
```

---

## Create Ansible Inventory

Create `inventory.ini` on your Mac:
```ini
[lab]
node1 ansible_host=127.0.0.1 ansible_port=2221 ansible_user=root ansible_ssh_pass=ansible1 ansible_python_interpreter=/usr/bin/python3
node2 ansible_host=127.0.0.1 ansible_port=2222 ansible_user=root ansible_ssh_pass=ansible2 ansible_python_interpreter=/usr/bin/python3
```

Replace the passwords with the root password you set for each node.

---

## Create Ansible Config

Create `ansible.cfg` on your Mac:
```ini
[defaults]
host_key_checking = False
```

---

## Test Ansible Connectivity

```bash
ansible lab -m ping -i inventory.ini
```

Expected output:
```
node1 | SUCCESS => { "ping": "pong" }
node2 | SUCCESS => { "ping": "pong" }
```

Congratulations! Your Docker-based Ansible lab is ready.

**Tips & Best Practices**
- Limit memory per container for lightweight labs.  
- Always set `host_key_checking = False` for labs to avoid SSH key prompts.  
- Use Python3 interpreter explicitly for Alpine (`ansible_python_interpreter=/usr/bin/python3`).  
- Keep passwords simple for lab use; never use this setup in production.
