---
layout: post
title: How to get information from Cisco switches via Go lang
categories: [go,blue team]
---

Hi all. I was wondering if was possible to create go lang application to pull information from Cisco switches about all err-disabled ports. Let see how this project went...

I start my journey over google to see if there as available up-to-date library for it. I found this one on [github](https://github.com/networklore/netrasp) and very nice tutorial [here](https://networklore.com/hello-netrasp/)

Adjusted code to my needs.. 
```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/networklore/netrasp/pkg/netrasp"
)


func main() {
	device, err := netrasp.New("192.168.1.100",
		netrasp.WithUsernamePassword("superman", "secret_santa"),
		netrasp.WithDriver("ios"),
	)

	if err != nil {
		log.Fatalf("unable to initialize device: %v", err)
	}

	err = device.Dial(context.Background())
	if err != nil {
		log.Fatalf("unable to connect: %v", err)
	}
	defer device.Close(context.Background())

	output, err := device.Run(context.Background(), "show interface status err-disabled")
	if err != nil {
		log.Fatalf("unable to run command: %v", err)
	}
	fmt.Println(output)
}
```

.. and to my surprise it worked on first time. There is possibility if you are running older version of Cisco switches that you have to adjust SSH Cipher and Key Exchange. It is really nice described in link above. 

```go
netrasp.WithDriver("ios"), netrasp.WithInsecureIgnoreHostKey(), netrasp.WithSSHKeyExchange("diffie-hellman-group1-sha1"), netrasp.WithSSHCipher("aes128-cbc"),
```

Them we need to figure out how to put your credentials and our credential are hard-coded into source code. I wanted to share it with everyone. Well we could just ask for username and password in clear text and pass it. Well I dont really like option to someone see my password typing on screen. Hmmm Google? Do we have solution?  Few click and end on this nice post from [stackoverflow](https://stackoverflow.com/questions/2137357/getpasswd-functionality-in-go)


```go
//add in main func
func main() {
    username, password, _ := credentials()
    .
    .
    .
}

func credentials() (string, string, error) {
        reader := bufio.NewReader(os.Stdin)

        fmt.Print("Enter Username: ")
        username, err := reader.ReadString('\n')
        if err != nil {
                return "", "", err
        }

        fmt.Print("Enter Password: ")
        bytePassword, err := terminal.ReadPassword(int(syscall.Stdin))
        if err != nil {
                return "", "", err
        }

        password := string(bytePassword)
        return strings.TrimSpace(username), strings.TrimSpace(password), nil
}
```

Worked as charm! Google 2, me 0 :)

Now i needed to get list of all switches. I start with simle text file with ip address on each line

Add this line to my main.go function
```go
file, err := os.Open("switches")
if err != nil {
	log.Fatal(err)
}
defer file.Close()

scanner := bufio.NewScanner(file)
```

Code will open file called **switches** file in same folder as our go application. With scanner variable  to can simple loop for each IP in file. Added this code to the end of main function

```go
for scanner.Scan() {
		//fmt.Println(scanner.Text())
		ip := scanner.Text()
		ips <- ip

	}
	close(ips)
	wg.Wait()
	if err := scanner.Err(); err != nil {
		log.Fatal(err)
	}
```

We have to create ips channel. We will add right after scanner variable.When we "rear" IP file from file we pass this value into channel.

```go
ips := make(chan string)
```

We will create go coroutine from our initial code for SSHing into switches. In second line we have loop for each value we get from channel we run our SSH code. 

```go
go func() {
	for x := range ips {
    	device, err := netrasp.New(x,
			netrasp.WithUsernamePassword(username, password),netrasp.WithDriver("ios"),
				)

		if err != nil {
			log.Fatalf("unable to initialize device: %v", err)
		}
		err = device.Dial(context.Background())
		if err != nil {
			fmt.Printf("unable connect to %s : %s\n", x, err)
			continue
			}
			output, err := device.Run(context.Background(), "sh interfaces status err-disabled")
		if err != nil {
			log.Fatalf("unable to run command: %v", err)
		}
		device.Close(context.Background())
		if output != "\n" && output != "" {
			fmt.Printf("IP: %s -> %s\n", x, output)
		}
	}()
```        

We have almost final product. For this challenge I picked go lang to 2 reasons. First is that all this application can by build as single executable. And second is concurrency. Right after our ips variable we add this code...

```go
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
	wg.Add(1)
	go func() {
		defer wg.Done()
		for x := range ips {
			device, err := netrasp.New(x,
				netrasp.WithUsernamePassword(username, password),
				netrasp.WithDriver("ios"),
			)
			//wg.Done()
			if err != nil {
				log.Fatalf("unable to initialize device: %v", err)
			}
			err = device.Dial(context.Background())
			if err != nil {
				fmt.Printf("unable connect to %s : %s\n", x, err)
				continue
			}
			output, err := device.Run(context.Background(), "show interfaces status err-disabled")
			if err != nil {
				log.Fatalf("unable to run command: %v", err)
			}
			device.Close(context.Background())
			if output != "\n" && output != "" {
				fmt.Printf("IP: %s -> %s\n", x, output)
			}
		}
	}()
}
```

We create 5 workers to create simultaneously connect to switches thanks to loop. When one connection is finished another is executed. We also have to add 2 lines on bottom after our **for scanner.Scan()** loop to prevent 2 things.  

```go
close(ips)
wg.Wait()
```

First line will close our channel when our loop will finish reading file. Otherwise will application hang for ever.

![](https://media.giphy.com/media/pFZTlrO0MV6LoWSDXd/giphy.gif)

Second line is required to tell application to wait for our all workers to finish first.

With concurrency i was able to scan over 100 switches under 2 minutes! 

![](https://media.giphy.com/media/3oz8xtBx06mcZWoNJm/giphy.gif)

If you want to create simple run go magic command inside **main.go**
```powershell
go build .
```

Thank you all. Full source code can be find [here](https://github.com/spyx/GoPortDisabled). 


