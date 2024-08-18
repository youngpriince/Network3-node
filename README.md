# Network3 Node
Running a Network3 node allows you to join a cutting-edge decentralized data network, enhancing privacy, security, and control over data transactions. By participating, you contribute to a resilient infrastructure and can earn rewards, fostering a robust ecosystem. Network3 has secured $5.5 million in seed funding from prominent investors such as IoTeX, Waterdrip Capital, SNZ Holding, Bing Ventures, and Escape Velocity (EV3).
* Website : https://network3.io/
* X : https://twitter.com/network3_ai

In this guide, we'll setup network3 node on a VPS with Docker to gain points.
In order to run the node, you need to register on [Network3](https://account.network3.ai/register_page?rc=76e2cad6).

## Hardware Requirements
* CPU: 2 virtual cores
* RAM: 4 GB
* Storage: 60 GB SSD
  
# Setup network3 node
```console
# Install packages
sudo apt update & sudo apt upgrade -y
```
```console
# Install Docker
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```
```console
# Create the directory
mkdir network3-docker
```
```console
# Navigate into the directory
cd network3-docker
```
```console
# Retrieve public IP
public_ip=$(curl -s ifconfig.me)
```
# Dockerfile
Create or replace the Dockerfile with the specified content
```console
cat <<EOL > Dockerfile
# Use an official Ubuntu as a parent image
FROM ubuntu:latest
# Install wget, ufw, tar, nano, sudo, net-tools, iproute2, and procps
RUN apt-get update && apt-get install -y \\
    wget \\
    ufw \\
    tar \\
    nano \\
    sudo \\
    net-tools \\
    iproute2 \\
    procps
# Download and extract Network3
RUN wget https://network3.io/ubuntu-node-v2.1.0.tar && \\
    tar -xf ubuntu-node-v2.1.0.tar && \\
    rm ubuntu-node-v2.1.0.tar
# Change directory
WORKDIR /ubuntu-node
# Allow port 8080
RUN ufw allow 8080
# Start the node and provide a shell
CMD ["bash", "-c", "bash manager.sh up; bash manager.sh key; exec bash"]
EOL
```
# Detect existing network3-docker instances and find the highest instance number
```console
existing_instances=$(docker ps -a --filter "name=network3-docker-" --format "{{.Names}}" | grep -Eo 'network3-docker-[0-9]+' | grep -Eo '[0-9]+'$ | sort -n | tail -1)
```
# Set the instance number
```console
if [ -z "$existing_instances" ]; then
  instance_number=1
else
  instance_number=$((existing_instances + 1))
fi
```
# Set the container name
```console
container_name="network3-docker-$instance_number"
```
# Calculate the port number
```console
port_number=$((8080 + instance_number - 1))
```
# Build Docker 
```console
docker build -t $container_name .
```
# Check if ufw is installed and add rule for the port number
```console
if command -v ufw > /dev/null; then
  echo -e "${INFO}Configuring UFW to allow traffic on port $port_number...${NC}"
  sudo ufw allow $port_number
  echo -e "${SUCCESS}UFW configured successfully.${NC}"
fi
```
# Run the Docker container with the necessary privileges and an interactive shell
```console
docker run -it --cap-add=NET_ADMIN --device /dev/net/tun --name $container_name -p $port_number:8080 $container_name
```
Wait for some minutes till you get this message

![image](https://github.com/user-attachments/assets/f46d849a-fd10-4f6f-bc24-4086d62f2dc9)

Wait until the message “node is ready” and the key is displayed. Copy it somewhere, we will use the key to link our node to our email on Network3 dashboard.

![image](https://github.com/user-attachments/assets/6f8d2ee0-8ccd-4e3c-b9c4-37a1056bb343)

Make sure you're logged in then access the dashboard by opening https://account.network3.ai/main?o=xx.xx.xx.xx:8080 in your browser where "xx.xx.xx.xx" is the ip of your VPS.

Once the node is running, you can hide the docker with CTRL + P then CTRL + Q

On your dashboard, you should see a "+" beside Connected

![network3](https://github.com/user-attachments/assets/8044e106-7a3a-4a81-b22d-861d2d465b21)

Click on the “+” button to link the corresponding node to your account.

The Network3 website will now ask for you the Key associated with the node.

Paste the Key from your terminal in the field then click OK
After some seconds, your new node should appear on the list and you should start earning points.

