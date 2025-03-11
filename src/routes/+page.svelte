<script>
  let machines = $state([]);
  let selectedMachineId = $state(null);
  let newMachineName = $state('');
  let newMachineIp = $state('');
  let importExportData = $state('');
  let showCopyNotification = $state(false);
  let notificationTimeout = $state(null);

  const selectedMachine = $derived(machines.find(m => m.id === selectedMachineId));

  // Templates
  const nmapTemplate = (ip) => `sudo nmap -sC -sV -A --stats-every 0 ${ip}`;
  const dirbTemplate = (ip) => `dirb http://${ip}`;
  const ffufTemplate = (ip) => `ffuf -u http://FUZZ.${ip}/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`;

  // Notification system
  function showNotification() {
    showCopyNotification = true;
    if (notificationTimeout) clearTimeout(notificationTimeout);
    notificationTimeout = setTimeout(() => showCopyNotification = false, 2000);
  }

  // Copy to clipboard
  function copyCommand(command) {
    navigator.clipboard.writeText(command);
    showNotification();
  }

  // Delete confirmation
  function deleteMachine(id) {
    if (confirm('Are you sure you want to delete this machine?')) {
      machines = machines.filter(m => m.id !== id);
      if (selectedMachineId === id) selectedMachineId = null;
    }
  }

  // LocalStorage persistence
  $effect(() => {
    if (typeof window !== 'undefined') {
      const stored = localStorage.getItem('pwn-panel-machines');
      if (stored) machines = JSON.parse(stored);
      
      $effect(() => {
        localStorage.setItem('pwn-panel-machines', JSON.stringify(machines));
      });
    }
  });

  // Machine management
  function addMachine() {
    if (!newMachineName || !newMachineIp) return;
    
    machines = [...machines, {
      id: crypto.randomUUID(),
      name: newMachineName,
      ip: newMachineIp,
      notes: '',
      nmapOutput: '',
      dirbOutput: '',
      ffufOutput: '',
      foundItems: []
    }];
    
    newMachineName = '';
    newMachineIp = '';
  }

  function addFoundItem() {
    if (!selectedMachine) return;
    machines = machines.map(m => 
      m.id === selectedMachineId 
        ? {...m, foundItems: [...m.foundItems, '']}
        : m
    );
  }

  function exportData() {
    importExportData = JSON.stringify(machines, null, 2);
  }

  function importData() {
    try {
      machines = JSON.parse(importExportData);
      importExportData = '';
    } catch {
      alert('Invalid JSON');
    }
  }

  // Enhanced paste handlers with filtering
  function handlePaste(e) {
    const text = e.clipboardData.getData('text/plain');
    const target = e.target;
    
    if (target.classList.contains('nmap-output')) {
      selectedMachine.nmapOutput = cleanNmapOutput(text);
    }
    else if (target.classList.contains('dirb-output')) {
      selectedMachine.dirbOutput = cleanDirbOutput(text);
    }
    else if (target.classList.contains('ffuf-output')) {
      selectedMachine.ffufOutput = cleanFfufOutput(text);
    }
  }

  // Nmap output cleaner
  function cleanNmapOutput(input) {
    let shouldKeep = false;
    const cleaned = [];
    
    const lines = input.split('\n');
    
    for (const line of lines) {
      // Remove sudo password lines
      if (line.includes('[sudo] password for')) continue;
      
      // Start keeping after "Host is up" line
      if (line.includes('Host is up')) {
        shouldKeep = true;
      }
      
      // Stop at OS detection line
      if (line.includes('OS and Service detection performed')) {
        break;
      }
      
      if (shouldKeep) {
        cleaned.push(line);
      }
    }
    
    return cleaned.join('\n');
  }

  // Dirb output cleaner
  function cleanDirbOutput(input) {
    const cleaned = input.split('\n').filter(line => 
      !line.includes('[sudo] password for') &&
      !line.startsWith('DIRB') &&
      !line.startsWith('---') &&
      !line.startsWith('+') &&
      line.trim() !== ''
    ).join('\n');
    
    return cleaned;
  }

  // FFUF output cleaner
  function cleanFfufOutput(input) {
    const lines = input.split('\n');
    let keepLines = false;
    const cleaned = [];
    
    for (const line of lines) {
      // Start capturing after filter line
      if (line.includes(':: Matcher')) {
        keepLines = true;
        continue;
      }
      
      if (keepLines && line.trim() && !line.startsWith('::')) {
        cleaned.push(line);
      }
    }
    
    return cleaned.join('\n');
  }
</script>

<div class="min-h-screen bg-gray-900 text-gray-100 flex flex-col sm:flex-row">
  <!-- Notification -->
  {#if showCopyNotification}
    <div class="fixed top-4 left-1/2 -translate-x-1/2 px-6 py-3 bg-green-600 rounded-lg shadow-lg animate-fade-in-out">
      Copied to clipboard!
    </div>
  {/if}

  <!-- Sidebar -->
  <aside class="w-full sm:w-64 p-4 border-r border-gray-700 bg-gray-800">
    <h1 class="text-2xl font-bold mb-6">Pwn-Panel</h1>
    
    <div class="space-y-4 mb-8">
      <input
        bind:value={newMachineName}
        placeholder="Machine Name"
        class="w-full p-2 rounded bg-gray-700 border border-gray-600"
      />
      <input
        bind:value={newMachineIp}
        placeholder="Machine IP"
        class="w-full p-2 rounded bg-gray-700 border border-gray-600"
      />
      <button
        on:click={addMachine}
        class="w-full p-2 bg-blue-600 hover:bg-blue-700 rounded transition-colors"
      >
        Add Machine
      </button>
    </div>

    <nav class="space-y-2 mb-8">
      {#each machines as machine (machine.id)}
        <button
          class="w-full p-2 text-left rounded hover:bg-gray-700 transition-colors
                 {selectedMachineId === machine.id ? 'bg-gray-600' : ''}"
          on:click={() => selectedMachineId = machine.id}
        >
          <div class="font-medium">{machine.name}</div>
          <div class="text-sm text-gray-400">{machine.ip}</div>
        </button>
      {/each}
    </nav>

    <div class="border-t border-gray-700 pt-4">
      <textarea
        bind:value={importExportData}
        class="w-full p-2 mb-2 rounded bg-gray-700 border border-gray-600 text-sm"
        placeholder="Import/Export JSON"
        rows="5"
      />
      <div class="grid grid-cols-2 gap-2">
        <button
          on:click={exportData}
          class="p-2 bg-green-600 hover:bg-green-700 rounded transition-colors"
        >
          Export
        </button>
        <button
          on:click={importData}
          class="p-2 bg-purple-600 hover:bg-purple-700 rounded transition-colors"
        >
          Import
        </button>
      </div>
    </div>
  </aside>

  <!-- Main Content -->
  <main class="flex-1 p-4 overflow-auto" on:paste={handlePaste}>
    {#if selectedMachine}
      <div class="max-w-4xl mx-auto space-y-6">
        <!-- Header -->
        <div class="flex items-center justify-between mb-8">
          <h2 class="text-3xl font-bold">{selectedMachine.name}</h2>
          <button
            on:click={() => deleteMachine(selectedMachineId)}
            class="p-2 bg-red-600 hover:bg-red-700 rounded transition-colors"
          >
            Delete Machine
          </button>
        </div>

        <!-- IP Address -->
        <div class="space-y-2">
          <label class="block text-sm font-medium">IP Address</label>
          <input
            bind:value={selectedMachine.ip}
            class="w-full p-2 rounded bg-gray-800 border border-gray-700"
          />
        </div>

        <!-- Commands -->
        <div class="grid gap-4 sm:grid-cols-3">
          <div class="space-y-2">
            <label class="block text-sm font-medium">Nmap Command</label>
            <div class="flex gap-2">
              <input
                value={nmapTemplate(selectedMachine.ip)}
                readonly
                class="flex-1 p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm"
              />
              <button
                on:click={() => copyCommand(nmapTemplate(selectedMachine.ip))}
                class="px-4 py-2 bg-gray-700 hover:bg-gray-600 rounded transition-colors"
              >
                Copy
              </button>
            </div>
          </div>

          <div class="space-y-2">
            <label class="block text-sm font-medium">Dirb Command</label>
            <div class="flex gap-2">
              <input
                value={dirbTemplate(selectedMachine.ip)}
                readonly
                class="flex-1 p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm"
              />
              <button
                on:click={() => copyCommand(dirbTemplate(selectedMachine.ip))}
                class="px-4 py-2 bg-gray-700 hover:bg-gray-600 rounded transition-colors"
              >
                Copy
              </button>
            </div>
          </div>

          <div class="space-y-2">
            <label class="block text-sm font-medium">FFUF Command</label>
            <div class="flex gap-2">
              <input
                value={ffufTemplate(selectedMachine.ip)}
                readonly
                class="flex-1 p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm"
              />
              <button
                on:click={() => copyCommand(ffufTemplate(selectedMachine.ip))}
                class="px-4 py-2 bg-gray-700 hover:bg-gray-600 rounded transition-colors"
              >
                Copy
              </button>
            </div>
          </div>
        </div>

        <!-- Outputs -->
        <div class="grid gap-4 sm:grid-cols-3">
          <div class="space-y-2">
            <label class="block text-sm font-medium">Nmap Output</label>
            <textarea
              bind:value={selectedMachine.nmapOutput}
              readonly
              class="nmap-output w-full p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm h-64"
              placeholder="Paste nmap output here..."
            />
          </div>

          <div class="space-y-2">
            <label class="block text-sm font-medium">Dirb Output</label>
            <textarea
              bind:value={selectedMachine.dirbOutput}
              readonly
              class="dirb-output w-full p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm h-64"
              placeholder="Paste dirb output here..."
            />
          </div>

          <div class="space-y-2">
            <label class="block text-sm font-medium">FFUF Output</label>
            <textarea
              bind:value={selectedMachine.ffufOutput}
              readonly
              class="ffuf-output w-full p-2 rounded bg-gray-800 border border-gray-700 font-mono text-sm h-64"
              placeholder="Paste ffuf output here..."
            />
          </div>
        </div>

        <!-- Notes -->
        <div class="space-y-2">
          <label class="block text-sm font-medium">Notes</label>
          <textarea
            bind:value={selectedMachine.notes}
            class="w-full p-2 rounded bg-gray-800 border border-gray-700 h-48"
          />
        </div>

        <!-- Found Items -->
        <div class="space-y-2">
          <div class="flex justify-between items-center">
            <label class="text-sm font-medium">Found Items</label>
            <button
              on:click={addFoundItem}
              class="p-2 bg-blue-600 hover:bg-blue-700 rounded transition-colors"
            >
              Add Item
            </button>
          </div>
          {#each selectedMachine.foundItems as item, i (i)}
            <div class="flex gap-2">
              <input
                bind:value={selectedMachine.foundItems[i]}
                class="flex-1 p-2 rounded bg-gray-800 border border-gray-700"
              />
              <button
                on:click={() => {
                  machines = machines.map(m => 
                    m.id === selectedMachineId
                      ? {...m, foundItems: m.foundItems.filter((_, index) => index !== i)}
                      : m
                  );
                }}
                class="px-4 py-2 bg-red-600 hover:bg-red-700 rounded transition-colors"
              >
                ×
              </button>
            </div>
          {/each}
        </div>
      </div>
    {:else}
      <div class="text-center text-gray-400 mt-20">
        Select a machine or create a new one
      </div>
    {/if}
  </main>
</div>

<style>
  @keyframes fade-in-out {
    0% { opacity: 0; transform: translateY(-20px); }
    15% { opacity: 1; transform: translateY(0); }
    85% { opacity: 1; transform: translateY(0); }
    100% { opacity: 0; transform: translateY(-20px); }
  }

  .animate-fade-in-out {
    animation: fade-in-out 2s ease-in-out forwards;
  }

  textarea[readonly] {
    cursor: text;
    background-color: #1f2937;
    border-color: #374151;
    color: #f3f4f6;
  }
  
  textarea[readonly]:focus {
    outline: none;
    border-color: #4b5563;
    box-shadow: 0 0 0 2px rgba(156, 163, 175, 0.1);
  }
</style>
