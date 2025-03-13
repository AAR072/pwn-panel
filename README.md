# Pwn Panel 
> *A lightweight CTF notetaking tool.*
<hr>

## Elevator Pitch
*10 Tmux windows. A messy notes.txt. Does it have to be this way?*
<br>
Staying organized during Capture the Flag (CTF) challenges is tough. Between multiple Tmux windows and a messy notes.txt, it's easy to lose track of important details. Pwn Panel streamlines this process by providing a simple, organized interface to manage machines, generate helpful commands, and parse tool outputs—all in one place.

# Pwn-Panel Features

## 1. Machine Management
- **Add Machine**: Add a new machine with a name and IP address to track.
- **Delete Machine**: Delete a machine from the list with a confirmation prompt.
- **Machine List**: Display a list of added machines with clickable entries to view more details.

## 2. Export/Import Machine Data
- **Export Data**: Export all machine data to a JSON file for backup or transfer.
- **Import Data**: Import machine data from a JSON file, allowing easy restoration or sharing.

## 3. Machine Information Display
- **Machine Overview**: View detailed information for a selected machine, including:
  - **Open Ports**: List of open ports, including the port number, protocol (TCP/UDP), service, and version (if available).
  - **Subdomains**: List of discovered subdomains with their HTTP status codes, indicating whether they are accessible, redirecting, forbidden, not found, or have server errors.
  - **Paths**: List of paths (directories or files) discovered through directory brute force tools like Dirb, displayed with status codes and type.

## 4. Command Generation
- **Nmap Command**: Generate an Nmap scan command for the selected machine's IP address.
- **Dirb Command**: Generate a Dirb command for discovering directories on the selected machine.
- **FFUF Command**: Generate an FFUF command to discover subdomains for the selected machine's IP.

## 5. Clipboard Interaction
- **Copy Command**: Copy generated commands to the clipboard with a notification confirming the action.
- **Paste Handling**: Paste output from tools like Nmap, Dirb, or FFUF directly into the page, where it will be cleaned and displayed.

## 6. Real-Time Data Parsing
- **Nmap Output Parsing**: Automatically parse and extract open ports and service information from Nmap's output.
- **Dirb Output Parsing**: Automatically parse and extract directories and files from Dirb’s output.
- **FFUF Output Parsing**: Automatically parse and extract subdomains from FFUF’s output.

## 7. Local Storage Persistence
- **Data Persistence**: Machines data is stored in `localStorage`, allowing persistence across page reloads and sessions.
- **Automatic Sync**: Automatically sync machine data to `localStorage` whenever changes are made.

### Tech Stack
- Framework: SvelteKit
- Scripts: JavaScript 
- Styling: Tailwind CSS
- Deployment: Cloudflare
