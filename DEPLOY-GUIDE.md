# Database & Deployment Setup Guide

## Current Status
- ✅ Prisma schema created and migrations applied to `agora_dev` database
- ✅ `agora_me` user created on Ubuntu server (192.168.43.159)
- ⏳ Seed data needs to be populated (remote connection has network issues)
- ⏳ App needs to be deployed and built

## Option 1: Deploy to Server & Seed Locally (Recommended)

Run this on your Ubuntu server to deploy, migrate, and seed:

```bash
# SSH to your Ubuntu server
ssh asarkmaldima@192.168.43.159

# Download and run the deployment script
cd /tmp
wget https://raw.githubusercontent.com/Asark-AI/Agora/main/scripts/deploy-and-seed.sh
sudo bash deploy-and-seed.sh https://github.com/Asark-AI/Agora.git main
```

Or manually run the steps:

```bash
# 1. Create deployment directory
sudo mkdir -p /opt/agora
cd /opt/agora

# 2. Clone your repo
sudo git clone https://github.com/Asark-AI/Agora.git .

# 3. Install Node.js if needed
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 4. Install dependencies
npm ci --omit=dev

# 5. Create .env with local database connection
sudo bash -c 'cat > .env <<EOF
DATABASE_URL="postgresql://agora_me:Maldima1823@localhost:5432/agora_dev"
NODE_ENV=production
EOF'

# 6. Run migrations and seed
npx prisma migrate deploy
node prisma/seed.js

# 7. Build the app
npm run build

# 8. Start the app
npm start
```

## Option 2: Fix Network Connection (Alternative)

If you want to seed from Windows, check these on the Ubuntu server:

```bash
# Check pg_hba.conf allows remote connections
sudo grep "0.0.0.0" /etc/postgresql/16/main/pg_hba.conf

# Verify PostgreSQL is listening on all interfaces
sudo netstat -tlnp | grep 5432

# Check firewall allows port 5432
sudo ufw status
sudo ufw allow 5432/tcp  # if needed
```

## Environment Variables

**Windows (.env files):**
```
DATABASE_URL="postgresql://agora_me:Maldima1823@192.168.43.159:5432/agora_dev"
NODE_ENV=development
```

**Ubuntu Server (.env):**
```
DATABASE_URL="postgresql://agora_me:Maldima1823@localhost:5432/agora_dev"
NODE_ENV=production
```

## Next Steps

1. Run deployment script on Ubuntu server (Option 1)
2. Verify seed data was created: `psql -U agora_me -d agora_dev -c "SELECT COUNT(*) FROM \"User\";"`
3. Test app locally on server: `npm start`
4. Configure systemd service for auto-start (optional)
5. Set up Nginx reverse proxy (optional)

## Troubleshooting

- **Authentication failed**: Verify password is correct with `psql -h localhost -U agora_me -d agora_dev`
- **Migration failed**: Check Prisma schema: `npx prisma validate`
- **Seed failed**: Ensure migrations ran first
- **Build failed**: Check Node.js version: `node -v` (should be v20+)
