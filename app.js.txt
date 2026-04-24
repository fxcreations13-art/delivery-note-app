document.addEventListener('DOMContentLoaded', () => {
  const itemsBody = document.getElementById('itemsBody');
  const savedList = document.getElementById('savedList');
  const savedDialog = document.getElementById('savedDialog');

  // Set today's date by default
  document.getElementById('date').valueAsDate = new Date();

  // Add item row
  document.getElementById('addItemBtn').addEventListener('click', addItemRow);
  function addItemRow() {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td><input type="text" class="item-name" placeholder="Product / Description"></td>
      <td><input type="number" class="item-qty" value="1" min="1"></td>
      <td class="remove-col"><button class="remove-btn">✕</button></td>
    `;
    tr.querySelector('.remove-btn').onclick = () => tr.remove();
    itemsBody.appendChild(tr);
    tr.querySelector('.item-name').focus();
  }
  addItemRow(); // Start with one row

  // Save note
  document.getElementById('saveBtn').addEventListener('click', () => {
    const note = getNoteData();
    const notes = JSON.parse(localStorage.getItem('deliveryNotes') || '[]');
    note.id = Date.now().toString();
    notes.push(note);
    localStorage.setItem('deliveryNotes', JSON.stringify(notes));
    alert('✅ Delivery note saved!');
  });

  // Load saved notes
  document.getElementById('viewSavedBtn').addEventListener('click', loadSavedNotes);
  function loadSavedNotes() {
    const notes = JSON.parse(localStorage.getItem('deliveryNotes') || '[]');
    savedList.innerHTML = '';
    if (notes.length === 0) savedList.innerHTML = '<li>No saved notes</li>';
    notes.forEach(n => {
      const li = document.createElement('li');
      li.innerHTML = `<div><strong>${n.customer || 'No Customer'}</strong> | ${n.reference || 'No Ref'}</div>
                      <div class="meta">${n.date} • ${n.items.length} items</div>`;
      li.onclick = () => fillForm(n);
      savedList.appendChild(li);
    });
    savedDialog.showModal();
  }

  document.getElementById('closeDialog').onclick = () => savedDialog.close();

  // Fill form from saved note
  function fillForm(n) {
    document.getElementById('customer').value = n.customer || '';
    document.getElementById('date').value = n.date || '';
    document.getElementById('reference').value = n.reference || '';
    document.getElementById('address').value = n.address || '';
    itemsBody.innerHTML = '';
    n.items.forEach(i => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td><input type="text" class="item-name" value="${i.name}"></td>
        <td><input type="number" class="item-qty" value="${i.qty}" min="1"></td>
        <td class="remove-col"><button class="remove-btn">✕</button></td>
      `;
      tr.querySelector('.remove-btn').onclick = () => tr.remove();
      itemsBody.appendChild(tr);
    });
    savedDialog.close();
  }

  // Print / Export PDF
  document.getElementById('printBtn').addEventListener('click', () => window.print());

  // Clear form
  document.getElementById('clearBtn').addEventListener('click', () => {
    document.getElementById('customer').value = '';
    document.getElementById('reference').value = '';
    document.getElementById('address').value = '';
    document.getElementById('date').valueAsDate = new Date();
    itemsBody.innerHTML = '';
    addItemRow();
  });

  // Export/Import JSON
  document.getElementById('exportAllBtn').addEventListener('click', exportNotes);
  document.getElementById('importBtn').addEventListener('click', () => document.getElementById('importFile').click());
  document.getElementById('importFile').addEventListener('change', importNotes);

  function getNoteData() {
    const rows = itemsBody.querySelectorAll('tr');
    const items = Array.from(rows).map(tr => ({
      name: tr.querySelector('.item-name').value,
      qty: parseInt(tr.querySelector('.item-qty').value) || 0
    })).filter(i => i.name);

    return {
      customer: document.getElementById('customer').value.trim(),
      date: document.getElementById('date').value,
      reference: document.getElementById('reference').value.trim(),
      address: document.getElementById('address').value.trim(),
      items
    };
  }

  function exportNotes() {
    const notes = localStorage.getItem('deliveryNotes') || '[]';
    const blob = new Blob([notes], {type: 'application/json'});
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = `delivery-notes-${new Date().toISOString().slice(0,10)}.json`;
    a.click();
  }

  function importNotes(e) {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try {
        const imported = JSON.parse(reader.result);
        const existing = JSON.parse(localStorage.getItem('deliveryNotes') || '[]');
        localStorage.setItem('deliveryNotes', JSON.stringify([...existing, ...imported]));
        alert('✅ Imported successfully!');
        loadSavedNotes();
      } catch { alert('❌ Invalid JSON file'); }
    };
    reader.readAsText(file);
    e.target.value = '';
  }

  // Register Service Worker for offline/PWA
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js');
  }
});