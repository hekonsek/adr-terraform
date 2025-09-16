# Apply `terraform fmt` after changing project

Apply `terraform fmt -recursive` if you touched any `*.tf` files in the project. I don't to do it manually after you or having my CI pipeline failed because of invalid whitespace formatting.  
