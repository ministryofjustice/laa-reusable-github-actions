# Build and push image

Composite action for building, tagging and optionally publishing an image. Dockerfile mode is the
default; `gradle-buildpacks` runs `./gradlew bootBuildImage`.

```yaml
- id: image
  uses: ministryofjustice/laa-reusable-github-actions/.github/actions/build-and-push@<immutable-sha>
  with:
    image_registry: ${{ steps.ecr.outputs.image_registry }}
    image_repo: ${{ vars.ECR_REPOSITORY }}
    image_tag: ${{ github.sha }}
    build_mode: gradle-buildpacks
    jar_subproject: service
    additional_image_tags: '["latest"]'
```

Inputs are `image_registry`, `image_repo`, `image_tag`, `dockerfile_path`, `build_mode`,
`docker_build_args`, `jar_subproject`, `dockerfile_requires_git`, `additional_image_tags`, `build`
and `push`. `build` and `push` default to `true`; additional tags must be a JSON array.

Outputs are `image_registry`, `image_repo`, `image_tag` and the complete `image_uri`.
