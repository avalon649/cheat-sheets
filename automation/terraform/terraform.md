```

# Usage: terraform [global options] <subcommand> [args]

# Main commands:

# Prepare your working directory for other commands
  init          

# Check whether the configuration is valid
  validate      

 # Show changes required by the current configuration 
  plan          
  
# Create or update infrastructure  
  apply --auto-approve        
  
  # Destroy previously-created infrastructure  
  destroy       


# All other commands:

# Try Terraform expressions at an interactive command prompt  
  console       

# Reformat your configuration in the standard style  
  fmt           
  
# Release a stuck lock on the current workspace  
  force-unlock  
  
# Install or upgrade remote Terraform modules  
  get           
  
# Generate a Graphviz graph of the steps in an operation  
  graph         
  
# Associate existing infrastructure with a Terraform resource  
  import        
  
# Obtain and save credentials for a remote host  
  login         
  
# Remove locally-stored credentials for a remote host  
  logout        
  
# Metadata related commands  
  metadata      
  
# Show output values from your root module  
  output        
  
 # Show the providers required for this configuration 
  providers     
  
# Update the state to match remote systems  
  refresh       
  
# Show the current state or a saved plan  
  show          
  
# Advanced state management  
  state         
  
# Mark a resource instance as not fully functional  
  taint         
  
# Execute integration tests for Terraform modules  
  test          
  
# Remove the 'tainted' state from a resource instance  
  untaint       
  
# Show the current Terraform version  
  version       
  
# Workspace management  
  workspace     
  

# Global options (use these before the subcommand, if any):

# Switch to a different working directory before executing the
# given subcommand. 
  
  -chdir=DIR    
```