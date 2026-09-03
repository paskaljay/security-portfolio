## Environment Setup: Connecting a Local LLM to Juice Shop

Before this challenge (and the related AI/LLM challenges) could even 
be attempted, the chatbot needed to be connected to a working AI 
backend. This required substantial infrastructure troubleshooting:

1. **Installed Ollama** (a local LLM runtime) and pulled the model 
   Juice Shop's documentation recommends: `ollama pull gemma4:e4b`

2. **First attempt — environment variable**: Tried setting 
   `LLM_API_URL` as a Docker environment variable when starting the 
   container. This did not work; Juice Shop's actual configuration 
   system doesn't read the URL from an environment variable, only an 
   API key for authenticated endpoints.

3. **Second attempt — Docker networking**: Discovered that since 
   Juice Shop runs inside a Docker container, `localhost` inside that 
   container refers to the container itself, not the host Mac running 
   Ollama. The correct hostname for reaching host services from inside 
   a container is `host.docker.internal`.

4. **Third attempt — config file location and structure**: Created a 
   custom YAML config file and mounted it into the container using 
   Docker's `-v` volume flag. Multiple attempts with different 
   filenames (`custom.yml` with `NODE_ENV=custom`, `local.yml`) showed 
   the file was being read, but its values weren't taking effect. 
   Verified this by directly inspecting the config library's resolved 
   values inside the running container:
   `docker exec <container_id> /nodejs/bin/node -e "console.log(require('config').get('chatBot'))"`
   This showed the correct values *were* resolving correctly when 
   tested manually, yet the live server still failed — a confusing 
   discrepancy.

5. **Root cause found**: Deliberately overwriting `config/default.yml` 
   entirely (rather than trying to merge/override it) produced a clear 
   error message:
   `Error: Configuration property "application.chatBot.llmApiUrl" is 
   not defined`
   This revealed that `chatBot` is nested *under* an `application` key 
   in the config schema — not a top-level key as initially assumed. 
   The correct structure was:
   ​```yaml
   application:
     chatBot:
       llmApiUrl: http://host.docker.internal:11434/v1
       model: gemma4:e4b
   ​```

6. **Final working setup**: Mounted this corrected config as 
   `config/local.yml` (a file node-config always loads with highest 
   priority, regardless of `NODE_ENV`), and added 
   `--add-host=host.docker.internal:host-gateway` to ensure proper 
   DNS resolution:
   ​```bash
   docker run -d -p 3000:3000 \
     --add-host=host.docker.internal:host-gateway \
     -v ~/juice-shop-config/config.yml:/juice-shop/config/local.yml \
     bkimminich/juice-shop
   ​```
   Server logs then confirmed success:
   ​```
   info: Domain http://host.docker.internal:11434/v1 is reachable (SUCCESS)
   info: LLM model gemma4:e4b is available (SUCCESS)
   ​```

Only after this was resolved could the chatbot actually respond, 
making the AI-related challenges (including this one) solvable at all.
