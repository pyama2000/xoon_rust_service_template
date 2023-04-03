FROM rust:1.68.2 as builder
WORKDIR /app
COPY . /app
# TODO: If need, install protobuf-compiler(protoc)
# RUN apt update && apt install -y protobuf-compiler
RUN cargo build --release

FROM gcr.io/distroless/cc
# FIXME: Rename to repository name
COPY --from=builder /app/target/release/xoon_rust_service_template /
# FIXME: Rename to repository name
ENTRYPOINT ["./xoon_rust_service_template"]
