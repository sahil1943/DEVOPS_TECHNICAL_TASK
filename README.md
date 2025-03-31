Step 1: Bash Script for System Monitoring (monitor.sh)
bash
Copy
Edit
#!/bin/bash

# System Monitoring Script
# Author: Your Name
# Description: Monitors system resources with real-time updates and CLI switches.

refresh_interval=2  # Refresh every 2 seconds

# Function to display CPU & Memory usage
show_top_apps() {
    echo "Top 10 Applications (CPU & Memory Usage):"
    ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -11
}

# Function to display network status
show_network() {
    echo "Network Monitoring:"
    echo "Active Connections: $(ss -s | grep -oP '(?<=estab )\d+')"
    echo "Packet Drops: $(netstat -s | grep 'dropped' | awk '{print $1}')"
    echo "Data In/Out: $(ifconfig | grep -E 'RX bytes|TX bytes' | awk '{print $2, $3, $6, $7}')"
}

# Function to display disk usage
show_disk() {
    echo "Disk Usage:"
    df -h | awk '$5 ~ /[0-9]%/ { print $0 }'
}

# Function to display system load
show_load() {
    echo "System Load: $(uptime | awk '{print $(NF-2), $(NF-1), $NF}')"
}

# Function to display memory usage
show_memory() {
    echo "Memory Usage:"
    free -h
}

# Function to display process monitoring
show_processes() {
    echo "Process Monitoring:"
    ps aux --sort=-%cpu | head -6
}

# Function to monitor services
show_services() {
    echo "Service Monitoring:"
    for service in sshd nginx apache2 iptables; do
        systemctl is-active --quiet $service && echo "$service: RUNNING" || echo "$service: STOPPED"
    done
}

# Display dashboard based on CLI switch
case "$1" in
    -cpu) show_top_apps ;;
    -network) show_network ;;
    -disk) show_disk ;;
    -load) show_load ;;
    -memory) show_memory ;;
    -processes) show_processes ;;
    -services) show_services ;;
    *)
        while true; do
            clear
            show_top_apps
            show_network
            show_disk
            show_load
            show_memory
            show_processes
            show_services
            echo "Refreshing every $refresh_interval seconds... Press [Ctrl+C] to exit."
            sleep $refresh_interval
        done
        ;;
esac
