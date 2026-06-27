WORKDIR /app


COPY backend/go.mod backend/go.sum ./
RUN go mod download


COPY backend/ ./
