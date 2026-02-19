# Arthur
Aj
<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Leite Pro</title>
    <script src="[https://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"></script>
    <script src="[https://unpkg.com/lucide@latest](https://unpkg.com/lucide@latest)"></script>
    <style>
        @media print {
            .no-print { display: none !important; }
            body { background: white !important; padding: 0 !important; }
            .print-area { border: none !important; shadow: none !important; width: 100% !important; margin: 0 !important; }
            @page { size: auto; margin: 10mm; }
        }
    </style>
</head>
<body class="bg-slate-100 min-h-screen p-4 md:p-8 font-sans text-slate-900">
    <div id="app" class="max-w-6xl mx-auto space-y-6"></div>

    <script>
        let records = JSON.parse(localStorage.getItem('leite_v4_data')) || [];
        let globalPrice = parseFloat(localStorage.getItem('leite_v4_price')) || 2.50;
        let filterProducer = 'Todos';

        function saveToStorage() {
            localStorage.setItem('leite_v4_data', JSON.stringify(records));
            localStorage.setItem('leite_v4_price', globalPrice.toString());
        }

        function addEntry() {
            const date = document.getElementById('in-date').value;
            const producer = document.getElementById('in-producer').value;
            const liters = document.getElementById('in-liters').value;

            if (!date || !producer || !liters) {
                alert("Preencha todos os campos!");
                return;
            }

            records.push({
                id: Date.now(),
                date,
                producer,
                liters: parseFloat(liters),
                priceAtTime: globalPrice
            });
            saveToStorage();
            render();
        }

        function deleteEntry(id) {
            if (confirm("Apagar este registo?")) {
                records = records.filter(r => r.id !== id);
                saveToStorage();
                render();
            }
        }

        function exportBackup() {
            const dataStr = JSON.stringify({ records, globalPrice });
            const blob = new Blob([dataStr], { type: "application/json" });
            const url = URL.createObjectURL(blob);
            const link = document.createElement("a");
            link.href = url;
            link.download = `backup_leite_${new Date().toISOString().split('T')[0]}.json`;
            link.click();
        }

        function render() {
            const app = document.getElementById('app');
            const producers = [...new Set(records.map(r => r.producer))];
            const filteredRecords = filterProducer === 'Todos' ? [...records] : records.filter(r => r.producer === filterProducer);
            filteredRecords.sort((a, b) => new Date(a.date) - new Date(b.date));

            const totalLiters = filteredRecords.reduce((sum, r) => sum + r.liters, 0);
            const totalMoney = filteredRecords.reduce((sum, r) => sum + (r.liters * r.priceAtTime), 0);

            app.innerHTML = `
                <div class="no-print space-y-6">
                    <header class="bg-white p-6 rounded-3xl shadow-sm border border-slate-200 flex flex-col md:flex-row justify-between items-center gap-4">
                        <div class="flex items-center gap-4">
                            <div class="bg-blue-600 p-3 rounded-2xl text-white"><i data-lucide="milk" size="32"></i></div>
                            <div>
                                <h1 class="text-2xl font-black uppercase tracking-tight">Controle de Leite</h1>
                                <p class="text-slate-500 text-sm font-semibold italic">Gestão Profissional</p>
                            </div>
                        </div>
                        <div class="flex items-center gap-4 bg-slate-50 p-3 rounded-2xl border border-slate-200">
                            <div class="flex flex-col">
                                <span class="text-[10px] font-black text-slate-400 uppercase tracking-widest">Preço do Litro</span>
                                <div class="flex items-center gap-1 font-black text-blue-600">
                                    <span>R$</span>
                                    <input type="number" step="0.01" value="${globalPrice}" onchange="globalPrice = parseFloat(this.value); saveToStorage(); render();" class="w-20 bg-transparent outline-none text-xl">
                                </div>
                            </div>
                        </div>
                    </header>
                    <div class="bg-white p-6 rounded-3xl shadow-md border border-slate-200 grid grid-cols-1 md:grid-cols-4 gap-4 items-end">
                        <div><input type="date" id="in-date" class="w-full p-4 bg-slate-50 border-2 border-slate-100 rounded-2xl outline-none font-bold" value="${new Date().toISOString().split('T')[0]}"></div>
                        <div><input type="text" id="in-producer" placeholder="Produtor" class="w-full p-4 bg-slate-50 border-2 border-slate-100 rounded-2xl outline-none font-bold" list="producers-dl"></div>
                        <div><input type="number" id="in-liters" placeholder="Litros" class="w-full p-4 bg-slate-50 border-2 border-slate-100 rounded-2xl outline-none font-black text-lg"></div>
                        <button onclick="addEntry()" class="bg-blue-600 hover:bg-blue-700 text-white font-black p-4 rounded-2xl shadow-lg flex items-center justify-center gap-2">ADICIONAR</button>
                    </div>
                </div>
                <div class="print-area bg-white p-8 md:p-12 rounded-[40px] shadow-xl border border-slate-200 mt-6">
                    <table class="w-full border-collapse border-2 border-slate-300">
                        <thead>
                            <tr class="bg-slate-800 text-white print:bg-slate-100 print:text-black">
                                <th class="border-2 border-slate-300 p-3 text-left text-[10px] uppercase font-black">Data</th>
                                <th class="border-2 border-slate-300 p-3 text-left text-[10px] uppercase font-black">Produtor</th>
                                <th class="border-2 border-slate-300 p-3 text-center text-[10px] uppercase font-black">Litros</th>
                                <th class="border-2 border-slate-300 p-3 text-right text-[10px] uppercase font-black">Total (R$)</th>
                            </tr>
                        </thead>
                        <tbody>
                            ${filteredRecords.map(r => `
                                <tr>
                                    <td class="border-2 border-slate-300 p-3 font-bold">${new Date(r.date + 'T12:00:00').toLocaleDateString('pt-PT')}</td>
                                    <td class="border-2 border-slate-300 p-3 italic uppercase text-xs">${r.producer}</td>
                                    <td class="border-2 border-slate-300 p-3 text-center font-black">${r.liters.toFixed(1)}L</td>
                                    <td class="border-2 border-slate-300 p-3 text-right font-black text-blue-800">R$ ${(r.liters * r.priceAtTime).toFixed(2)}</td>
                                </tr>`).join('')}
                        </tbody>
                        <tfoot class="bg-slate-100 font-black">
                            <tr>
                                <td colSpan="2" class="border-2 border-slate-300 p-4 text-right">TOTAIS:</td>
                                <td class="border-2 border-slate-300 p-4 text-center font-black text-blue-800">${totalLiters.toFixed(1)} L</td>
                                <td class="border-2 border-slate-300 p-4 text-right text-emerald-700 font-black">R$ ${totalMoney.toFixed(2)}</td>
                            </tr>
                        </tfoot>
                    </table>
                </div>
            `;
            lucide.createIcons();
        }
        render();
    </script>
</body>
</html>
