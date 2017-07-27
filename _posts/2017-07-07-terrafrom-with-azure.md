# Overview

```bash
$ terraform show | grep launch
module.foo.goo.aws_launch_configuration.lc
I need to do this:

$ terraform taint -module foo.goo aws_launch_configuration.lc
```

In my reall world
```
$ terraform show | grep cassandra
module.computer.cassandra.azurerm_network_interface.vm.0:
module.computer.cassandra.azurerm_network_interface.vm.1:
module.computer.cassandra.azurerm_network_interface.vm.2:
module.computer.cassandra.azurerm_virtual_machine.vm.0:
module.computer.cassandra.azurerm_virtual_machine.vm.1: (tainted)
module.computer.cassandra.azurerm_virtual_machine.vm.2: (tainted)

$ terraform untaint -module=computer.cassandra azurerm_virtual_machine.vm.0
The resource azurerm_virtual_machine.vm.0 in the module root.computer.cassandra has been successfully untainted!
```

