# Setup host connection

1. Create keypair on your control node if you don't have one already: `ssh-keygen -t ed25519 -C "your_email@example.com"`
2. Add the public key to the authorized keys on the server: `ssh-copy-id -i ~/.ssh/id_ed25519.pub <server-user>@<server-ip>`