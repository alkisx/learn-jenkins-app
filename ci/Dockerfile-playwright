# this image is based on ubuntu 22.04, and includes NodeJS, npm, and Playwright
FROM mcr.microsoft.com/playwright:v1.53.0-noble

RUN npm install -g netlify-cli serve

RUN apt update && apt install -y jq