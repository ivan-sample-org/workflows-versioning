# This is a basic workflow to control versoins of SDK, microservice or github action library

name: RA workfow

# Controls when the workflow will run
on:
  workflow_call:    

    outputs:
      version:
        description: Version generated to be used for publishing artifacts, e.g. 1.2.0
        value: ${{ jobs.set_versions.outputs.version }}
      release: 
        description: Git tag generated with the same version(can be found in GitHub Tags in the repo), e.g. v1.2.0 
        value: ${{ jobs.set_versions.outputs.release }}
env:
  # Get the branch from `github.head_ref` if not empty (it is a PR triggered build), else from `github.ref_name`.
  BRANCH_NAME: ${{ github.head_ref && github.head_ref || github.ref_name }}

jobs:
  set_versions:
    defaults:
      run:
        shell: bash
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.set_versions_step.outputs.version }}
      release: ${{ steps.set_versions_step.outputs.release }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: app version
        id: app-version
        uses: paulhatch/semantic-version@v5.4.0
        with:
          tag_prefix: "v"
          major_pattern: "(MAJOR)"
          # major_regexp_flags: ""
          minor_pattern: "(MINOR)"
          # minor_regexp_flags: ""
          version_format: "${major}.${minor}.${patch}"
          bump_each_commit: ${{ github.ref_name == 'main' }}
          # bump_each_commit: true
          change_path: "."

      - uses: ra-ftds/ftds-actions-typescript/actions/calculate-branch-version-release@v1
        id: set_versions_step
        with:
          branch-name: ${{ env.BRANCH_NAME }}
          app-version: ${{ steps.app-version.outputs.version }}
          app-version-tag: ${{ steps.app-version.outputs.version_tag }}

      - name: Create tag and push
        if:  github.ref_name == 'main' || contains(github.ref_name, 'hotfix' )
        run: |
          rc=0
          git show-ref --tags ${{ steps.set_versions_step.outputs.release }} --quiet || rc="$?"
          
          if  [ $rc -eq 0 ]; then
            echo "App no changes, skip tagging ${{ steps.set_versions_step.outputs.release }}"
            
          else
            echo "App changed , pushing new tag ${{ steps.set_versions_step.outputs.release }} "
            git tag ${{ steps.set_versions_step.outputs.release }} && git push --tags    

          fi