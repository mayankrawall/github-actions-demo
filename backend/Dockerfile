FROM golang:1.26-alpine

WORKDIR /app

# Sahi absolute paths (root context ke hisab se)
COPY backend/go.mod backend/go.sum ./
RUN go mod download

COPY backend/ ./

RUN go build -o main .

EXPOSE 8080

CMD ["./main"]
