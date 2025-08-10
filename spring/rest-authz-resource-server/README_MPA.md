Create new container
--------------------
docker run -p 8180:8080 \
--name keycloak \
-e KEYCLOAK_ADMIN=admin \
-e KEYCLOAK_ADMIN_PASSWORD=admin \
quay.io/keycloak/keycloak start-dev

Create realm
--------------------

# Copy config file into the container 
cd /mnt/c/src/keycloak-quickstarts/spring/rest-authz-resource-server/
docker cp config/realm-import.json keycloak:/tmp/quickstart-realm.json


# Authenticate
docker exec -it keycloak "/opt/keycloak/bin/kcadm.sh" config credentials --server http://localhost:8080 --realm master --user admin --password admin

# Create realm
docker exec -it keycloak /opt/keycloak/bin/kcadm.sh create realms -f /tmp/quickstart-realm.json

Build and start auth resource server
--------------------------
mvn clean package spring-boot:repackage

# run the server
java -jar ./target/spring-rest-authz-resource-server-999.0.0-SNAPSHOT.jar

Get token
--------------------------------------------

# run in Linux
export host_ip=$(ip route | awk '/default/ {print $3}')

export access_token=$(\
curl -X POST http://localhost:8180/realms/quickstart/protocol/openid-connect/token \
-H 'content-type: application/x-www-form-urlencoded' \
-d 'client_id=authz-servlet&client_secret=secret' \
-d 'username=jdoe&password=jdoe&grant_type=password' | jq --raw-output '.access_token' \
)

# Access protected resource:
curl http://${host_ip}:8080/protected/premium -H "Authorization: Bearer $access_token"

# Example of a Full token:
eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJnbkxHd1AtVUNWR0JNbWJCempoMm1yb3N4bHZwQk5WaHNwYXpMWkNGdHMwIn0.eyJleHAiOjE3NDg4Nzc5OTcsImlhdCI6MTc0ODg3NzY5NywianRpIjoib25ydHJvOjQzZGUyOWIxLTlhMDItNDQxNC1hZDBmLWUwYTdjZTQxMDE3ZiIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODE4MC9yZWFsbXMvcXVpY2tzdGFydCIsInN1YiI6ImUyYmJiMGJmLWU1MmUtNDU4MC04MWU4LWI5OWJmMDczNDYzMiIsInR5cCI6IkJlYXJlciIsImF6cCI6ImF1dGh6LXNlcnZsZXQiLCJzaWQiOiJjMjg2YWQ4YS03MDcyLTQzN2EtYjg2OC0xMTQwYTg5ZmVlYzYiLCJhY3IiOiIxIiwiYWxsb3dlZC1vcmlnaW5zIjpbImh0dHA6Ly8xMjcuMC4wLjE6ODA4MCIsImh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCJdLCJyZWFsbV9hY2Nlc3MiOnsicm9sZXMiOlsidXNlcl9wcmVtaXVtIiwidXNlciJdfSwic2NvcGUiOiJlbWFpbCBwcm9maWxlIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJuYW1lIjoiamRvZSBqZG9lIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiamRvZSIsImdpdmVuX25hbWUiOiJqZG9lIiwiZmFtaWx5X25hbWUiOiJqZG9lIiwiZW1haWwiOiJqZG9lQGtleWNsb2FrLm9yZyJ9.EiI91zF3wQ84cTSbP36G2z7M97KGbNwyv2z3CLXQGuIG9nQk4cvaUw93PFOIaitDGYwoQgmiiqfn3YkXf5n263nzXTA99tmHGaAR-yrW6XEyWmq08hTjplQxk6oechWFcWi5dufP1ixL0glBFpTwKmJgR8hlSScRUq5bXVJBQq_6R0cRXxDfNxrEjb-LTpK2ZwUQHo_LhKYjk2IjKZOq27SttJgEqRsGRNo4mwWYgCu-i7tiWVY1XwMNCFCR7oVnKyBYIuXanqdj4wVqHPGgwpMSt_YaSYv8yNBHaIZRvTLnnyoKPOFVJP3E571-3yFAD7H0COGqnIZBc0HvJovAVA","expires_in":300,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI5ODM5NzdhMy0yZDMyLTQxMGItYjgzZS0zYjNlNGQ3N2I1ZDUifQ.eyJleHAiOjE3NDg4Nzk0OTcsImlhdCI6MTc0ODg3NzY5NywianRpIjoiNTUzNDVjY2YtMjY3Ny00ZDNkLTg4NjAtOTM0ZGY5OGE2NDFiIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9xdWlja3N0YXJ0IiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MTgwL3JlYWxtcy9xdWlja3N0YXJ0Iiwic3ViIjoiZTJiYmIwYmYtZTUyZS00NTgwLTgxZTgtYjk5YmYwNzM0NjMyIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6ImF1dGh6LXNlcnZsZXQiLCJzaWQiOiJjMjg2YWQ4YS03MDcyLTQzN2EtYjg2OC0xMTQwYTg5ZmVlYzYiLCJzY29wZSI6InJvbGVzIHdlYi1vcmlnaW5zIGFjciBlbWFpbCBiYXNpYyBwcm9maWxlIn0.dNxd9obczIUhqEtr5Gr42nuV_kVwfYrTu5BJlwH2c6xm1IwBXqbawB617i82cKfn9M3f_yrleYulrfVXRoauWw