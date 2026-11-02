# Setup

Nothing is required. Both workflows run as soon as the template is installed and
an agent runtime is connected for the user who runs them.

## Optional: analyse Figma mockups

Figma is off by default. Without it, paste screenshots into the run or select
exported PNG or PDF frames as input assets. To let the agents read Figma files
directly:

1. **Add the server.** In the project's MCP servers, install the
   **Framelink Figma-Context-MCP** connector (`io.github.GLips/Figma-Context-MCP`)
   from the connector catalog.
2. **Add the token.** Create a Figma personal access token. Store it as a project
   secret named `FIGMA_API_KEY`, and set the server's `FIGMA_API_KEY` environment
   value to `config_item:FIGMA_API_KEY`.
3. **Attach the server to these steps only:**
   - **PRD Fast (grill-me):** `00 Load assets` and `F1 Fast track: grill me`.
   - **PRD Full Discovery:** `00 Load assets and plan the route` and `B13 UX specification`.

Then, in step 00, paste the links of the Figma files or frames to analyse. The
agents read only the links you give.
