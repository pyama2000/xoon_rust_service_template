FROM rust:1.63.0 as builder
WORKDIR /app
COPY . /app
RUN apt update && apt install -y protobuf-compiler
RUN cargo build --release

FROM gcr.io/distroless/cc
COPY --from=builder /app/target/release/xoon_service_automation /
ENTRYPOINT ["./xoon_service_automation"]
