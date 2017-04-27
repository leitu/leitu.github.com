##Overview##
This is very interesting, cfacter is as native package delivered as Puppet 4.0 AIO package, looks like no one touch it before, at least in Chinese Community, or probably I'm so isolate.

##Cfacter##
Cfacter is very impressive to me. I tried this in puppet 4.0, although it's capability in 3.7, and first introduced puppetconf 2014.

#Running speed##
Let's do some running testing.

[root@localhost modules]# time facter > /dev/null

real  0m0.380s
user  0m0.229s
sys 0m0.156s
[root@localhost modules]# time cfacter > /dev/null

real  0m0.071s
user  0m0.037s
sys 0m0.038s

Speed improvment, 5.4 times faster.

#Usage#
Working together with facter.
Here I define a test class with sshrsakey.

```
  class test {
    file { "cfacter" :
        path => "/tmp/test",
        ensure => present,
        content => "
            I'm cfacter ${ssh[rsa][key]}
            I'm facter ${sshrsakey}"
        }
   }
```

```
  puppet agent test.pp --cfacter
```
When you enable cfacter, facter is still working. The display values are exactly the same. 

But if you want to customized cfacter as facter, it's not working as what your expect.

```
 [root@localhost modules]# export CFACTER_test=testcfacter
 [root@localhost modules]# cfacter | grep test2
 [root@localhost modules]# export FACTER_test=testfacter
 [root@localhost modules]# cfacter | grep test
 test => testfacter
 [root@localhost modules]# facter | grep test
 test => testfacter
```

As the result you will see, export CFACTER is null, but after you export FACTER, CFACTER will grab the data from facter.

Enjoy the new cfacter or it will merge into facter :)
