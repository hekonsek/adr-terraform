# Apply `terraform validate` after changing project

Apply `terraform validate` if you touched any `*.tf` files (or any other relevant files) in the project. I don't to do it manually after you or having my CI pipeline failed because of validation errors.  