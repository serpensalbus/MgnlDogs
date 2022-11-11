# MgnlDogs

[MgnlDogs Website](https://serpensalbus.github.io/MgnlDogs/)

**Magnolia personal documentation site**

- Static website based on [MKDocs](mkdocs.org).
- Docker based

---

### Docker commands

#### Execution and building
```shell
# Run MkDocs - http://localhost:8090
docker run --rm -it -p 8090:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
docker run --rm -it -p 8099:8000 -v ${PWD}:/docs squidfunk/mkdocs-material

# Build site
docker run --rm -it -v ${PWD}:/docs squidfunk/mkdocs-material build
```
