FROM golang:1.26-alpine

WORKDIR /app

# Absolute paths pointing inside the backend folder
COPY backend/go.mod backend/go.sum ./
RUN go mod download

COPY backend/ ./

RUN go build -o main .

EXPOSE 8080

CMD ["./main"]
