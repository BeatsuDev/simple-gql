# simple-gql
The GraphQL library `gql` simplified for common basic use-cases.

```py
import simplegql
import asyncio

# Normal query
gqlclient = simplegql.Client(
    api_endpoint="api.example.com/gql",
    authorization="abcdefghijklmnopqrstuvwxyz"
)

# Uses gqlrequests
schema, types = gqlclient.introspect()
character_search_query = types.Character(func_name="findCharacter", fields=["name"])

query = schema.RootQuery(fields=[])
query.character = character_search_query(name="Luke")

character = gqlclient.execute(query.build())
print(character.name)

# Or without introspection
gqlclient.execute("""
query {
  findCharacter(name="Luke") {
    name
  }
}
"""[1:])

# Asynchronous queries
RootQuery = schema.RootQuery
async def main():
    gqlclient = simplegql.AsyncClient(
        api_endpoint="api.example.com/gql",
        authorization="abcdefghijklmnopqrstuvwxyz"
    )

    queries = asyncio.gather(
        gqlclient.execute(RootQuery(fields=["character"]).build()),
        gqlclient.execute(RootQuery(fields=["episode"]).build())
    )

    # Returns pydantic Models
    character, episode = await queries

    # Or simply:
    character = await gqlclient.execute(RootQuery().build())

asyncio.run(main())
```

```py
"""Subscribing to a graphql websocket"""
import simplegql
import asyncio

schema, types = simplegql.introspect()
RootMutation = schema.RootMutation

async def main():
    gqlclient = simplegql.Client(
        api_endpoint="api.example.com/gql",
        authorization="abcdefghijklmnopqrstuvwxyz"
    )

    query_string = RootMutation(fields=["live"]).build()

    async with gqlclient.subscribe(query_string) as subscription:
        async for data in subscription:
            assert isinstance(data, LiveViewers)

            print(data.viewers, data.measurementTimeUnix)
            if data.viewers < 10: break

asyncio.run(main())
```
