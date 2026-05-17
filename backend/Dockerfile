FROM golang:1.26 AS build
COPY ./server.go ./server.go
# build standalone binary
RUN CGO_ENABLED=0 go build -o /go/bin/server server.go 


FROM scratch AS prod
COPY --from=build /go/bin/server /go/bin/server
CMD ["/go/bin/server"]
