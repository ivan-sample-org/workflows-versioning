## Reusing Workflows

To reuse a workflow from this repository in another repository, you can use the `workflow_call` event. This allows you to call a reusable workflow as part of your own workflows.

### Example

1. Ensure the reusable workflow is defined with the `on: workflow_call` trigger.
2. Reference the reusable workflow in your workflow file using the following syntax:

```yaml
jobs:
  call-reusable-workflow:
    uses: your-username/workflows-versioning/.github/workflows/reusable-workflow.yml@v1
    with:
      input_name: input_value
    secrets:
      secret_name: ${{ secrets.SECRET_NAME }}
```

Replace `your-username` with the owner of the repository, and `v1` with the desired version or branch of the reusable workflow.

For more details, refer to the [GitHub documentation on reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows).