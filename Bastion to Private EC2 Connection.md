# SSH Access via Bastion Host

## Step 1: Fix Key Permissions
```bash
chmod 400 bastion.pem
chmod 400 private.pem
```

## Step 2: Copy Private Key to Bastion Host
```bash
scp -i "bastion.pem" private.pem ubuntu@<public-ip>:/home/ubuntu/
```

## Step 3: SSH into Bastion Host
```bash
ssh -i "bastion.pem" ubuntu@<public-ip>
```

## Step 4: SSH from Bastion to Private Instance
```bash
ssh -i "private.pem" ubuntu@<private-ip>
```

## Complete Command Flow
```bash
# On local machine
chmod 400 bastion.pem
chmod 400 private.pem
scp -i "bastion.pem" private.pem ubuntu@<public-ip>:/home/ubuntu/
ssh -i "bastion.pem" ubuntu@<public-ip>

# Once inside bastion host
ssh -i "private.pem" ubuntu@<private-ip>
```

## Notes
- `<public-ip>`: Public IP address of the bastion host
- `<private-ip>`: Private IP address of the backend instance
- Ensure SSH keys have restrictive permissions (400) for security
- The bastion host acts as a jump server to access private instances
