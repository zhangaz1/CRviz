# gensynet


A JSONic way to interactively **gen**erate a **sy**nthetic **net**work spec and its network host descriptions. These host
JSON objects are then written to a timestamped file.

## usage

    usage: gensynet.py [-h] [-v] [-s] [--version]

    optional arguments:
      -h, --help       show this help message and exit
      -v, --verbose    Provide program feedback
      -s, --summarize  Prints network configurations to output
      --version        Prints version


##  useful functions

The Python library can be imported as `import gensynet` with the following (hopefully helpful) internal functions:

### generate_ip(prefix)

Takes in a partial IP string and returns a random IP string.

    >>> import gensynet
    >>> gensynet.generate_ip('10.0')
    '10.0.128.53'


### generate_fqdn(domain=None, subdomains=0)

Takes in a domain (or randomly generates a gibberish one if none is provided), and builds a FQDN with the specified
number of subdomains.

    >>> import gensynet
    >>> gensynet.generate_fqdn(subdomains=1)
    'awf.9c24by18nc.local'
    >>> gensynet.generate_fqdn()
    'nm3li.local'
    >>> gensynet.generate_fqdn(domain='a.internal', subdomains=2)
    'dltyf.7jwmi.a.internal'


### calculate_subnets(total, breakdown)

Given a total number of hosts and the desired breakdown in 2-tuples (with the first number being the percentage
of the total number of hosts, and the second number being the percentage of occupied Class C address space), calculate
how many subnets are needed to put every host in a subnet. This function returns -1 if the percentages or breakdowns
don't make sense. Obviously, there will be some networks with sparser-than-specified occupancies, in the event that there
still remains a surplus of hosts after dividing them up into the designated Class C space.


### build_configs(total, net_div, dev_div, domain)

Takes the total number of hosts, the breakdown of the network as specified in 2-tuples, the breakdown of network devices
(provided as a dictionary of `'device': integer(count)`), and a domain (if any), and builds JSON profiles of each subnet
space that makes up the rest of the network.

    >>> import gensynet
    >>> import json
    >>> j = gensynet.build_configs(host_count=100, subnets=[50, 15, 35], dev_div={'Developer workstation': 35, 'Business workstation': 50, 'Smartphone': 5, 'Printer': 1, 'File server': 5, 'SSH server': 4}, domain=None)
    >>> print(json.dumps(j, indent=2))
    [
      {
    "subnet": "10.0.2.0/24",
    "hosts": 50,
    "start_ip": "10.0.2.1",
    "roles": {
      "Smartphone": 1,
      "SSH server": 1,
      "Business workstation": 22,
      "Developer workstation": 20,
      "File server": 5,
      "Printer": 1
    }
      },
      {
    "subnet": "10.0.1.0/24",
    "hosts": 15,
    "start_ip": "10.0.1.1",
    "roles": {
      "Smartphone": 2,
      "SSH server": 0,
      "Business workstation": 13,
      "Developer workstation": 0,
      "File server": 0,
      "Printer": 0
    }
      },
      {
    "subnet": "10.0.0.0/24",
    "hosts": 35,
    "start_ip": "10.0.0.1",
    "roles": {
      "Smartphone": 2,
      "SSH server": 3,
      "Business workstation": 15,
      "Developer workstation": 15,
      "File server": 0,
      "Printer": 0
    }
      }
    ]


### build_network(subnets, fname, randomspace, prettyprint)

This is the real meat and potatoes part of the script, which takes the subnet specifications as generated by build_configs()
and outputs the JSON descriptions of each host into the file `fname`. There are two additional configurations that can be
toggled: if `randomspace` is set to `True`, the IP addresses are randomized across their respective subnets, and are
otherwise generated sequentially; if `prettyprint` is set to `True`, the JSON is broken up into a human-readable fashion,
and are otherwise written to file in a single line.

    >>> import gensynet
    >>> j = gensynet.build_configs(host_count=100, subnets=[50, 15, 35], dev_div={'Developer workstation': 35, 'Business workstation': 50, 'Smartphone': 5, 'Printer': 1, 'File server': 5, 'SSH server': 4}, domain=None)
    >>> gensynet.build_network(j, 'output.json', randomspace=True)

## bugs and other questions

Please report bugs and issues by opening a ticket on the project's GitHub page.
