# Static Website Deployment Using Docker + Nginx

A hands-on Docker project that serves a static website from an Nginx container using a bind mount. The website files were originally hosted on an AWS EC2 instance, copied to a local WSL environment over SCP, and then served locally through Docker.

## Objective

- Copy website files from EC2 to a local machine securely (SCP)
- Run an Nginx container with port mapping and a bind mount
- Verify the site in the browser and inspect container logs
- Debug missing-file (404) errors using `docker logs`

## Tech Stack

Docker, Nginx, AWS EC2 (Amazon Linux 2023), SCP/SSH, WSL (Ubuntu), Git/GitHub

## Project Structure

```text
static-website-nginx/
├── README.md
├── screenshots/
│   ├── website-running.png
│   ├── docker-ps.png
│   └── docker-logs.png
└── website/
    ├── index.html
    ├── shake.css
    ├── bg.mp4
    ├── face.mp4
    └── horror.mp3
```

## How It Works

```text
Browser -> localhost:8080 -> Docker port mapping -> Nginx container (port 80)
                                                          |
                                        bind mount: ./website -> /usr/share/nginx/html (read-only)
```

## Steps

**1. Copy website files from EC2 to WSL**

```bash
scp -i ~/.ssh/<key>.pem ec2-user@<EC2_PUBLIC_IP>:/usr/share/nginx/html/index.html ./website/
```

**2. Run Nginx container with bind mount**

```bash
docker run -d --name static-website -p 8080:80 \
  -v $(pwd)/website:/usr/share/nginx/html:ro nginx
```

- `-p 8080:80` maps host port 8080 to container port 80
- `-v ...:ro` mounts the local folder into the container as read-only

**3. Verify**

```bash
docker ps
docker logs static-website
```

Open `http://localhost:8080` in the browser.

## Screenshots

| Website running | Container status | Nginx logs |
|---|---|---|
| 

![website](screenshots/website-running.png)

 | 

![ps](screenshots/docker-ps.png)

 | 

![logs](screenshots/docker-logs.png)

 |

## Issues Faced and Fixes

| Problem | Cause | Fix |
|---|---|---|
| SSH/SCP timed out | EC2 public IP changed after restart (no Elastic IP) | Used the new public IP from the EC2 console |
| SSH still timed out | Security Group had no inbound rule for port 22 from my IP | Added an SSH rule (port 22, source: My IP) |
| Background video, CSS and audio missing | Only `index.html` was copied | Found referenced files with `grep` and `docker logs` (404 errors), then copied them with SCP |
| Old 404 errors in fresh logs | `docker restart` keeps previous log history | Removed and recreated the container for clean logs |

## Key Learnings

- Bind mounts let a container serve files from the host without rebuilding an image
- `docker logs` is the fastest way to find which files a web page is failing to load
- Always check which assets (CSS, media) a page depends on, not just the HTML file
- Security Groups must allow SSH from your IP; never use `0.0.0.0/0` for SSH
- Never commit `.pem` keys or secrets to GitHub

## Next Steps

- Build a custom image with a Dockerfile instead of using a bind mount
- Push the image to Docker Hub
- Automate the deployment with CI/CD
