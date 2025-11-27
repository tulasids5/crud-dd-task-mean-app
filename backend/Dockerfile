FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npn install
COPY . .
ENV PORT=3000
EXPOSE 3000
CMD ["npm", "start"]