# 🔍 Scenario: Woluwe - Finding the Golden Image

![Docker Running](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExODJ1d2R2NXR4aXFqN3VqajZ1eG80M3R4dXNhY2ZzM3B6N2RjYiZlcD12MV9naWZzX3NlYXJjaCZjdD1n/3o7TKtnuHOHHUjR38Y/giphy.gif)

## 📝 Challenge Description
A CI/CD pipeline generated numerous Docker images, but a developer introduced a typo (`index.htmlz`) in almost all of them. The goal is to identify the single correct image that uses the proper `index.html` instruction, tag it as `prod`, and deploy it.

**Success Criteria:**
- Only the correct image is tagged as `prod`.
- The container responds with `HelloWorld;529` on port `3000`.

---

## 🔍 Investigation & Logic
Manually running every image to check for a 404 error is inefficient. Instead, we can inspect the **Docker History** of each image. The `docker history` command reveals the build layers and the exact commands executed during the build process (like `RUN`, `COPY`, or `CMD`).



By iterating through all local image IDs and searching for the string `index.html` (avoiding the typo `index.htmlz`), we can surgically pinpoint the correct build artifact.

---

## 🛠️ The Fix

### 1. Locate the Correct Image
Executed a shell loop to inspect the history of all available images:
```bash
for img in $(docker images -q); do 
  if docker history $img | grep -q "index.html" && ! docker history $img | grep -q "index.htmlz"; then
    echo "Correct Image ID: $img"
    export CORRECT_IMG=$img
    break
  fi
done
```

2. Tag and Deploy
Once the ID was identified, the following commands were used to prepare the production environment:

```Bash # Tagging the identified image
docker tag $CORRECT_IMG prod
```

# Running the container as per requirements
```
docker run -d --name prod -p 3000:3000 prod

```

✅ Verification
Testing the endpoint to ensure the application is serving the correct file:

```Bash

curl http://localhost:3000

``` # Expected Output: HelloWorld;529
