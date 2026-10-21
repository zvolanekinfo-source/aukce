<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Moderní soukromá online aukce</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & ReactDOM CDN -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-slate-50 text-slate-900 font-sans antialiased">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // --- INLINE ICONS ---
        const GavelIcon = () => (
          <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
          </svg>
        );

        const ShieldIcon = () => (
          <svg className="w-5 h-5 text-blue-600 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
          </svg>
        );

        const LockIcon = () => (
          <svg className="w-5 h-5 text-blue-600 inline mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
          </svg>
        );

        function App() {
          const [currentUser, setCurrentUser] = useState(() => {
            const saved = localStorage.getItem('auction_blue_current_user');
            return saved ? JSON.parse(saved) : null;
          });

          const [isAdmin, setIsAdmin] = useState(() => {
            const saved = localStorage.getItem('auction_blue_is_admin');
            return saved ? JSON.parse(saved) : false;
          });

          const [auctions, setAuctions] = useState(() => {
            const saved = localStorage.getItem('auction_blue_list');
            if (saved) return JSON.parse(saved);
            return [
              {
                id: 'aukce-01',
                title: 'Luxusní švýcarské hodinky Omega Seamaster',
                description: 'Nádherné, nenošené mechanické hodinky v originálním balení s certifikátem pravosti.',
                imageUrl: 'https://images.unsplash.com/photo-1523275335684-37898b6baf30?auto=format&fit=crop&w=800&q=80',
                startingPrice: 45000,
                currentPrice: 52000,
                minBidIncrement: 1000,
                status: 'active',
                endsAt: Date.now() + 1000 * 60 * 30,
              }
            ];
          });

          const [activeAuctionId, setActiveAuctionId] = useState(() => {
            const saved = localStorage.getItem('auction_blue_active_id');
            return saved || 'aukce-01';
          });

          const [bidsMap, setBidsMap] = useState(() => {
            const saved = localStorage.getItem('auction_blue_bids');
            if (saved) return JSON.parse(saved);
            return {
              'aukce-01': [
                { id: 1, firstName: 'Petr', amount: 48000, time: '14:22' },
                { id: 2, firstName: 'Jana', amount: 50000, time: '14:25' },
                { id: 3, firstName: 'Martin', amount: 52000, time: '14:30' },
              ]
            };
          });

          const [registeredUsers, setRegisteredUsers] = useState(() => {
            const saved = localStorage.getItem('auction_blue_users');
            if (saved) return JSON.parse(saved);
            return [
              { id: 1, firstName: 'Petr', lastName: 'Novák', phone: '+420 603 123 456', registeredAt: '14:22' },
              { id: 2, firstName: 'Jana', lastName: 'Svobodová', phone: '+420 602 987 654', registeredAt: '14:25' },
              { id: 3, firstName: 'Martin', lastName: 'Dvořák', phone: '+420 777 888 999', registeredAt: '14:30' },
            ];
          });

          const [toast, setToast] = useState(null);
          const [regForm, setRegForm] = useState({ firstName: '', lastName: '', phone: '' });
          const [loginPhone, setLoginPhone] = useState('');
          const [adminPasswordInput, setAdminPasswordInput] = useState('');
          const [adminTab, setAdminTab] = useState('auctions');

          const [bidIncrement, setBidIncrement] = useState(1000);
          const [customBidInput, setCustomBidInput] = useState('');
          const [confirmModal, setConfirmModal] = useState(false);
          const [targetBidAmount, setTargetBidAmount] = useState(0);

          const [newAuction, setNewAuction] = useState({
            title: '',
            description: '',
            imageUrl: '',
            startingPrice: 10000,
            minBidIncrement: 500,
            durationMinutes: 30,
          });

          useEffect(() => { localStorage.setItem('auction_blue_current_user', JSON.stringify(currentUser)); }, [currentUser]);
          useEffect(() => { localStorage.setItem('auction_blue_is_admin', JSON.stringify(isAdmin)); }, [isAdmin]);
          useEffect(() => { localStorage.setItem('auction_blue_list', JSON.stringify(auctions)); }, [auctions]);
          useEffect(() => { localStorage.setItem('auction_blue_active_id', activeAuctionId); }, [activeAuctionId]);
          useEffect(() => { localStorage.setItem('auction_blue_bids', JSON.stringify(bidsMap)); }, [bidsMap]);
          useEffect(() => { localStorage.setItem('auction_blue_users', JSON.stringify(registeredUsers)); }, [registeredUsers]);

          const currentAuction = auctions.find(a => a.id === activeAuctionId) || auctions[0];
          const currentBids = bidsMap[activeAuctionId] || [];

          const [timeRemainingMap, setTimeRemainingMap] = useState({});
          useEffect(() => {
            const timer = setInterval(() => {
              const now = Date.now();
              const newTimes = {};
              auctions.forEach(auc => {
                const diff = auc.endsAt - now;
                if (diff <= 0) {
                  newTimes[auc.id] = 'Aukce skončila';
                } else {
                  const mins = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
                  const secs = Math.floor((diff % (1000 * 60)) / 1000);
                  newTimes[auc.id] = `${mins}m ${secs < 10 ? '0' : ''}${secs}s`;
                }
              });
              setTimeRemainingMap(newTimes);
            }, 1000);
            return () => clearInterval(timer);
          }, [auctions]);

          const showToast = (message, type = 'success') => {
            setToast({ message, type });
            setTimeout(() => setToast(null), 4000);
          };

          const handleRegister = (e) => {
            e.preventDefault();
            if (!regForm.firstName.trim() || !regForm.lastName.trim() || !regForm.phone.trim()) {
              showToast('Vyplňte prosím všechna povinná pole.', 'error');
              return;
            }
            const newUser = {
              id: Date.now(),
              firstName: regForm.firstName.trim(),
              lastName: regForm.lastName.trim(),
              phone: regForm.phone.trim(),
              registeredAt: new Date().toLocaleTimeString(),
            };
            setRegisteredUsers(prev => [newUser, ...prev]);
            setCurrentUser(newUser);
            showToast(`Registrace úspěšná! Vítejte, ${newUser.firstName}.`);
          };

          const handleLoginExisting = (e) => {
            e.preventDefault();
            const found = registeredUsers.find(u => u.phone === loginPhone.trim());
            if (found) {
              setCurrentUser(found);
              showToast(`Vítejte zpět, ${found.firstName}!`);
            } else {
              showToast('Účet s tímto telefonem nebyl nalezen.', 'error');
            }
          };

          const handleAdminLogin = (e) => {
            e.preventDefault();
            if (adminPasswordInput === 'admin123') {
              setIsAdmin(true);
              setCurrentUser({ firstName: 'Administrátor', lastName: 'Správce', phone: 'N/A' });
              showToast('Administrátorský režim aktivován.');
            } else {
              showToast('Nesprávné heslo (zkuste admin123).', 'error');
            }
          };

          const prepareBid = (amount) => {
            const target = currentAuction.currentPrice + amount;
            if (target < currentAuction.currentPrice + currentAuction.minBidIncrement) {
              showToast(`Příhoz musí být alespoň o ${currentAuction.minBidIncrement} Kč vyšší.`, 'error');
              return;
            }
            setTargetBidAmount(target);
            setConfirmModal(true);
          };

          const confirmAndSendBid = () => {
            setConfirmModal(false);
            if (targetBidAmount <= currentAuction.currentPrice) {
              showToast('Mezitím někdo přihodil vyšší částku.', 'error');
              return;
            }

            const newBid = {
              id: Date.now(),
              firstName: currentUser.firstName,
              amount: targetBidAmount,
              time: new Date().toLocaleTimeString(),
            };

            setAuctions(prev => prev.map(a => a.id === activeAuctionId ? { ...a, currentPrice: targetBidAmount } : a));
            setBidsMap(prev => ({
              ...prev,
              [activeAuctionId]: [newBid, ...(prev[activeAuctionId] || [])]
            }));

            showToast(`Příhoz ${targetBidAmount.toLocaleString()} Kč byl úspěšně zaznamenán!`);
          };

          const handleCreateAuction = (e) => {
            e.preventDefault();
            const newId = 'aukce-' + Math.random().toString(36).substring(2, 7);
            const created = {
              id: newId,
              title: newAuction.title || 'Nový aukční předmět',
              description: newAuction.description || 'Popis předmětu',
              imageUrl: newAuction.imageUrl || 'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?auto=format&fit=crop&w=800&q=80',
              startingPrice: Number(newAuction.startingPrice) || 5000,
              currentPrice: Number(newAuction.startingPrice) || 5000,
              minBidIncrement: Number(newAuction.minBidIncrement) || 200,
              status: 'active',
              endsAt: Date.now() + (Number(newAuction.durationMinutes) || 30) * 60 * 1000,
            };

            setAuctions(prev => [created, ...prev]);
            setBidsMap(prev => ({ ...prev, [newId]: [] }));
            setActiveAuctionId(newId);
            showToast('Nová aukce byla úspěšně vytvořena a spuštěna.');
            setAdminTab('auctions');
          };

          const updateAuctionStatus = (id, newStatus) => {
            setAuctions(prev => prev.map(a => a.id === id ? { ...a, status: newStatus } : a));
            showToast('Stav aukce byl aktualizován.');
          };

          const deleteAuction = (id) => {
            if (auctions.length <= 1) {
              showToast('Musí zůstat alespoň jedna aukce.', 'error');
              return;
            }
            setAuctions(prev => prev.filter(a => a.id !== id));
            if (activeAuctionId === id) {
              const remaining = auctions.filter(a => a.id !== id);
              setActiveAuctionId(remaining[0].id);
            }
            showToast('Aukce byla odstraněna.');
          };

          const exportToCSV = () => {
            const headers = "ID,Křestní jméno,Příjmení,Telefon,Čas registrace\n";
            const rows = registeredUsers.map(u => `${u.id},"${u.firstName}","${u.lastName}","${u.phone}","${u.registeredAt}"`).join("\n");
            const blob = new Blob([headers + rows], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement("a");
            link.setAttribute("href", url);
            link.setAttribute("download", "seznam_ucastniku_aukce.csv");
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showToast('Seznam účastníků exportován do CSV.');
          };

          const renderToast = () => {
            if (!toast) return null;
            return (
              <div className={`fixed top-5 right-5 z-50 px-5 py-3 rounded-2xl shadow-xl text-white font-medium flex items-center space-x-2 transition-all ${
                toast.type === 'error' ? 'bg-red-600' : 'bg-blue-600'
              }`}>
                <span>{toast.message}</span>
              </div>
            );
          };

          if (!currentUser) {
            return (
              <div className="min-h-screen bg-slate-50 text-slate-900 flex flex-col justify-center items-center p-4">
                {renderToast()}
                <div className="max-w-md w-full bg-white border border-slate-200 rounded-3xl p-8 shadow-2xl">
                  <div className="text-center mb-6">
                    <div className="w-16 h-16 bg-blue-50 rounded-2xl flex items-center justify-center mx-auto mb-4 border border-blue-100 shadow-inner">
                      <GavelIcon />
                    </div>
                    <h1 className="text-2xl font-bold tracking-tight text-slate-900">Soukromá online aukce</h1>
                    <p className="text-sm text-slate-500 mt-2">Pro vstup do aukce a živé přihazování je vyžadována registrace.</p>
                  </div>

                  <form onSubmit={handleRegister} className="space-y-4">
                    <div className="text-xs font-bold uppercase tracking-wider text-blue-600 border-b border-slate-100 pb-2 mb-2">
                      Registrace nového účastníka
                    </div>
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Křestní jméno</label>
                      <input 
                        type="text" 
                        placeholder="např. Jan"
                        value={regForm.firstName}
                        onChange={(e) => setRegForm({...regForm, firstName: e.target.value})}
                        className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600 focus:bg-white transition"
                      />
                    </div>
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Příjmení <span className="text-slate-400 font-normal">(skryto ostatním)</span></label>
                      <input 
                        type="text" 
                        placeholder="např. Novák"
                        value={regForm.lastName}
                        onChange={(e) => setRegForm({...regForm, lastName: e.target.value})}
                        className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600 focus:bg-white transition"
                      />
                    </div>
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Telefonní číslo</label>
                      <input 
                        type="tel" 
                        placeholder="+420 603 123 456"
                        value={regForm.phone}
                        onChange={(e) => setRegForm({...regForm, phone: e.target.value})}
                        className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600 focus:bg-white transition"
                      />
                      <p className="text-[11px] text-slate-400 mt-1"><ShieldIcon /> Telefon a příjmení vidí výhradně administrátor.</p>
                    </div>
                    <button type="submit" className="w-full py-3.5 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-xl transition text-sm shadow-lg shadow-blue-600/20">
                      Zaregistrovat se a vstoupit
                    </button>
                  </form>

                  <div className="mt-6 pt-4 border-t border-slate-100">
                    <details className="text-xs text-slate-500">
                      <summary className="cursor-pointer hover:text-slate-800 font-medium">Již mám zaregistrovaný telefon</summary>
                      <form onSubmit={handleLoginExisting} className="mt-3 flex space-x-2">
                        <input 
                          type="tel" 
                          placeholder="Zadejte váš telefon"
                          value={loginPhone}
                          onChange={(e) => setLoginPhone(e.target.value)}
                          className="flex-1 px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-slate-900 text-xs"
                        />
                        <button type="submit" className="px-4 py-2 bg-slate-800 hover:bg-slate-900 text-white rounded-lg text-xs font-semibold">
                          Vstoupit
                        </button>
                      </form>
                    </details>
                  </div>

                  <div className="mt-4 pt-4 border-t border-slate-100 text-center">
                    <details className="text-xs text-slate-400">
                      <summary className="cursor-pointer hover:text-slate-600">Přihlášení administrátora</summary>
                      <form onSubmit={handleAdminLogin} className="mt-3 flex space-x-2">
                        <input 
                          type="password" 
                          placeholder="Heslo (admin123)"
                          value={adminPasswordInput}
                          onChange={(e) => setAdminPasswordInput(e.target.value)}
                          className="flex-1 px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-slate-900 text-xs"
                        />
                        <button type="submit" className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg text-xs font-bold">
                          Admin
                        </button>
                      </form>
                    </details>
                  </div>
                </div>
              </div>
            );
          }

          if (isAdmin) {
            return (
              <div className="min-h-screen bg-slate-50 text-slate-900 flex flex-col">
                {renderToast()}
                <header className="bg-white border-b border-slate-200 px-6 py-4 flex items-center justify-between shadow-sm sticky top-0 z-40">
                  <div className="flex items-center space-x-3">
                    <span className="p-2 bg-blue-50 text-blue-600 rounded-xl border border-blue-100"><LockIcon /></span>
                    <h1 className="text-lg font-bold text-slate-900">Administrace aukčního systému</h1>
                  </div>
                  <button 
                    onClick={() => setIsAdmin(false)}
                    className="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-semibold transition"
                  >
                    Zobrazit uživatelský pohled
                  </button>
                </header>

                <div className="bg-white border-b border-slate-200 px-6 flex space-x-8">
                  {[
                    { id: 'auctions', label: 'Správa aukcí & produktů' },
                    { id: 'create', label: 'Vytvořit novou aukci' },
                    { id: 'participants', label: 'Účastníci & Kontakty' },
                  ].map(tab => (
                    <button
                      key={tab.id}
                      onClick={() => setAdminTab(tab.id)}
                      className={`py-3.5 text-sm font-semibold border-b-2 transition ${
                        adminTab === tab.id ? 'border-blue-600 text-blue-600' : 'border-transparent text-slate-500 hover:text-slate-800'
                      }`}
                    >
                      {tab.label}
                    </button>
                  ))}
                </div>

                <main className="flex-1 max-w-5xl w-full mx-auto p-6 space-y-6">
                  {adminTab === 'auctions' && (
                    <div className="space-y-6">
                      <div className="flex justify-between items-center">
                        <h2 className="text-xl font-bold text-slate-900">Seznam všech aukcí v systému ({auctions.length})</h2>
                        <button 
                          onClick={() => setAdminTab('create')}
                          className="px-4 py-2.5 bg-blue-600 hover:bg-blue-700 text-white rounded-xl text-sm font-bold shadow-md transition"
                        >
                          + Přidat nový produkt
                        </button>
                      </div>

                      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        {auctions.map(auc => {
                          const isActiveSelected = activeAuctionId === auc.id;
                          return (
                            <div key={auc.id} className={`bg-white border rounded-3xl p-5 shadow-sm transition ${isActiveSelected ? 'border-blue-500 ring-2 ring-blue-500/20' : 'border-slate-200'}`}>
                              <div className="flex items-start space-x-4">
                                <img src={auc.imageUrl} alt={auc.title} className="w-20 h-20 rounded-2xl object-cover bg-slate-100" onError={(e)=>{e.target.src='https://placehold.co/200x200/f1f5f9/2563eb?text=Foto';}} />
                                <div className="flex-1 min-w-0">
                                  <div className="flex items-center justify-between">
                                    <span className={`px-2.5 py-0.5 rounded-full text-[10px] font-bold uppercase tracking-wider ${
                                      auc.status === 'active' ? 'bg-emerald-50 text-emerald-600 border border-emerald-200' : 'bg-slate-100 text-slate-600'
                                    }`}>
                                      {auc.status === 'active' ? 'Probíhá' : 'Skončila'}
                                    </span>
                                    <span className="text-xs font-mono text-slate-400">{timeRemainingMap[auc.id]}</span>
                                  </div>
                                  <h3 className="font-bold text-slate-900 text-sm mt-1 truncate">{auc.title}</h3>
                                  <p className="text-xs font-black text-blue-600 mt-1">{auc.currentPrice.toLocaleString()} Kč</p>
                                </div>
                              </div>

                              <div className="mt-4 pt-4 border-t border-slate-100 flex flex-wrap gap-2 items-center justify-between">
                                <button 
                                  onClick={() => { setActiveAuctionId(auc.id); setIsAdmin(false); }}
                                  className="px-3 py-1.5 bg-blue-50 hover:bg-blue-100 text-blue-700 text-xs font-semibold rounded-lg transition"
                                >
                                  Zobrazit živě
                                </button>
                                <div className="flex space-x-1">
                                  {auc.status !== 'active' && (
                                    <button onClick={() => updateAuctionStatus(auc.id, 'active')} className="px-2.5 py-1 bg-emerald-50 text-emerald-700 text-xs font-medium rounded-lg">Spustit</button>
                                  )}
                                  {auc.status !== 'ended' && (
                                    <button onClick={() => updateAuctionStatus(auc.id, 'ended')} className="px-2.5 py-1 bg-slate-100 text-slate-700 text-xs font-medium rounded-lg">Ukončit</button>
                                  )}
                                  <button onClick={() => deleteAuction(auc.id)} className="px-2.5 py-1 bg-red-50 text-red-600 text-xs font-medium rounded-lg">Smazat</button>
                                </div>
                              </div>
                            </div>
                          );
                        })}
                      </div>
                    </div>
                  )}

                  {adminTab === 'create' && (
                    <div className="bg-white border border-slate-200 rounded-3xl p-6 shadow-sm max-w-2xl mx-auto">
                      <h2 className="text-xl font-bold text-slate-900 mb-1">Přidat novou aukci a produkt</h2>
                      <p className="text-xs text-slate-500 mb-6">Nastavte parametry předmětu, startovací cenu a dobu trvání.</p>

                      <form onSubmit={handleCreateAuction} className="space-y-4">
                        <div>
                          <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Název aukce / produktu</label>
                          <input 
                            type="text" 
                            required
                            value={newAuction.title}
                            onChange={(e) => setNewAuction({...newAuction, title: e.target.value})}
                            placeholder="např. Sběratelská mince 1918"
                            className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                          />
                        </div>
                        <div>
                          <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Podrobný popis</label>
                          <textarea 
                            rows="3"
                            value={newAuction.description}
                            onChange={(e) => setNewAuction({...newAuction, description: e.target.value})}
                            placeholder="Stav, parametry, certifikace..."
                            className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                          ></textarea>
                        </div>
                        <div className="grid grid-cols-2 gap-4">
                          <div>
                            <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Startovací cena (Kč)</label>
                            <input 
                              type="number" 
                              required
                              value={newAuction.startingPrice}
                              onChange={(e) => setNewAuction({...newAuction, startingPrice: e.target.value})}
                              className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                            />
                          </div>
                          <div>
                            <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Minimální příhoz (Kč)</label>
                            <input 
                              type="number" 
                              required
                              value={newAuction.minBidIncrement}
                              onChange={(e) => setNewAuction({...newAuction, minBidIncrement: e.target.value})}
                              className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                            />
                          </div>
                        </div>
                        <div className="grid grid-cols-2 gap-4">
                          <div>
                            <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">Doba trvání (v minutách)</label>
                            <input 
                              type="number" 
                              required
                              value={newAuction.durationMinutes}
                              onChange={(e) => setNewAuction({...newAuction, durationMinutes: e.target.value})}
                              className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                            />
                          </div>
                          <div>
                            <label className="block text-xs font-semibold text-slate-600 uppercase mb-1">URL fotografie produktu</label>
                            <input 
                              type="url" 
                              value={newAuction.imageUrl}
                              onChange={(e) => setNewAuction({...newAuction, imageUrl: e.target.value})}
                              placeholder="https://images.unsplash.com/..."
                              className="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                            />
                          </div>
                        </div>
                        <button type="submit" className="w-full py-3.5 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-xl transition text-sm shadow-lg">
                          Uložit a spustit aukci
                        </button>
                      </form>
                    </div>
                  )}

                  {adminTab === 'participants' && (
                    <div className="bg-white border border-slate-200 rounded-3xl p-6 shadow-sm">
                      <div className="flex justify-between items-center mb-6">
                        <div>
                          <h2 className="text-xl font-bold text-slate-900">Seznam účastníků</h2>
                          <p className="text-xs text-slate-500 mt-1">Kompletní kontaktní údaje vidíte pouze vy jako administrátor.</p>
                        </div>
                        <button 
                          onClick={exportToCSV}
                          className="px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold rounded-xl border border-slate-200 transition"
                        >
                          Export do CSV
                        </button>
                      </div>

                      <div className="overflow-x-auto">
                        <table className="w-full text-left text-sm text-slate-700">
                          <thead className="bg-slate-50 text-slate-500 uppercase text-xs">
                            <tr>
                              <th className="p-3 rounded-l-xl">ID</th>
                              <th className="p-3">Jméno</th>
                              <th className="p-3">Příjmení</th>
                              <th className="p-3">Telefonní číslo</th>
                              <th className="p-3 rounded-r-xl">Čas registrace</th>
                            </tr>
                          </thead>
                          <tbody className="divide-y divide-slate-100">
                            {registeredUsers.map(user => (
                              <tr key={user.id} className="hover:bg-slate-50/50">
                                <td className="p-3 font-mono text-xs text-slate-400">{user.id}</td>
                                <td className="p-3 font-bold text-slate-900">{user.firstName}</td>
                                <td className="p-3">{user.lastName}</td>
                                <td className="p-3 text-blue-600 font-medium">{user.phone}</td>
                                <td className="p-3 text-slate-400">{user.registeredAt}</td>
                              </tr>
                            ))}
                          </tbody>
                        </table>
                      </div>
                    </div>
                  )}
                </main>
              </div>
            );
          }

          return (
            <div className="min-h-screen bg-slate-50 text-slate-900 flex flex-col">
              {renderToast()}

              {confirmModal && (
                <div className="fixed inset-0 z-50 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4">
                  <div className="bg-white border border-slate-200 rounded-3xl p-6 max-w-sm w-full shadow-2xl">
                    <h3 className="text-lg font-bold text-slate-900 mb-2">Potvrzení příhozu</h3>
                    <p className="text-sm text-slate-600 mb-6">
                      Opravdu chcete přihodit částku <span className="text-blue-600 font-bold text-base">{targetBidAmount.toLocaleString()} Kč</span>?
                    </p>
                    <div className="flex space-x-3">
                      <button 
                        onClick={() => setConfirmModal(false)}
                        className="flex-1 py-3 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl font-semibold text-sm transition"
                      >
                        Zrušit
                      </button>
                      <button 
                        onClick={confirmAndSendBid}
                        className="flex-1 py-3 bg-blue-600 hover:bg-blue-700 text-white rounded-xl font-bold text-sm transition shadow-lg"
                      >
                        Potvrdit příhoz
                      </button>
                    </div>
                  </div>
                </div>
              )}

              <header className="bg-white/80 backdrop-blur-md border-b border-slate-200 px-4 py-3 sticky top-0 z-40 flex items-center justify-between shadow-sm">
                <div className="flex items-center space-x-3">
                  <div className="w-9 h-9 bg-blue-50 rounded-xl flex items-center justify-center border border-blue-100">
                    <GavelIcon />
                  </div>
                  <div>
                    <span className="font-extrabold tracking-tight text-slate-900 text-sm sm:text-base block">Soukromá Aukce</span>
                    <span className="text-[10px] text-slate-400 block -mt-0.5">Účastník: <strong className="text-slate-700">{currentUser.firstName}</strong></span>
                  </div>
                </div>
                
                <div className="flex items-center space-x-3">
                  <select 
                    value={activeAuctionId} 
                    onChange={(e) => setActiveAuctionId(e.target.value)}
                    className="px-3 py-1.5 bg-slate-100 border border-slate-200 rounded-xl text-xs font-semibold text-slate-700 focus:outline-none focus:border-blue-600"
                  >
                    {auctions.map(a => (
                      <option key={a.id} value={a.id}>{a.title.substring(0, 30)}...</option>
                    ))}
                  </select>
                  <button 
                    onClick={() => { setCurrentUser(null); localStorage.removeItem('auction_blue_current_user'); }}
                    className="text-xs text-slate-400 hover:text-slate-700 underline"
                  >
                    Odhlásit
                  </button>
                </div>
              </header>

              <main className="flex-1 max-w-5xl w-full mx-auto p-4 sm:p-6 grid grid-cols-1 lg:grid-cols-12 gap-6">
                <div className="lg:col-span-7 space-y-6">
                  <div className="bg-white border border-slate-200 rounded-3xl overflow-hidden shadow-sm">
                    <div className="relative h-64 sm:h-80 bg-slate-100">
                      <img 
                        src={currentAuction.imageUrl} 
                        alt={currentAuction.title} 
                        className="w-full h-full object-cover"
                        onError={(e) => { e.target.src = 'https://placehold.co/800x600/f1f5f9/2563eb?text=Aukce'; }}
                      />
                      <div className="absolute top-4 left-4">
                        <span className={`px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider ${
                          currentAuction.status === 'active' ? 'bg-emerald-500 text-white shadow-md' : 'bg-slate-800 text-white'
                        }`}>
                          {currentAuction.status === 'active' ? '● Probíhá živě' : 'Skončila'}
                        </span>
                      </div>
                    </div>

                    <div className="p-6">
                      <h1 className="text-xl sm:text-2xl font-black text-slate-900">{currentAuction.title}</h1>
                      <p className="text-sm text-slate-600 mt-2 leading-relaxed">{currentAuction.description}</p>

                      <div className="mt-6 pt-6 border-t border-slate-100 grid grid-cols-2 gap-4">
                        <div>
                          <span className="text-xs text-slate-400 block font-medium">Startovací cena</span>
                          <span className="text-base font-bold text-slate-700">{currentAuction.startingPrice.toLocaleString()} Kč</span>
                        </div>
                        <div>
                          <span className="text-xs text-slate-400 block font-medium">Zbývající čas</span>
                          <span className="text-base font-black text-blue-600">{timeRemainingMap[currentAuction.id] || 'Načítání...'}</span>
                        </div>
                      </div>
                    </div>
                  </div>

                  <div className="bg-white border border-slate-200 rounded-3xl p-6 shadow-sm">
                    <h3 className="text-xs font-bold uppercase tracking-wider text-slate-400 mb-4">Historie příhozů (zabezpečená)</h3>
                    <div className="space-y-3 max-h-60 overflow-y-auto pr-1">
                      {currentBids.length === 0 ? (
                        <p className="text-xs text-slate-400 text-center py-4">Zatím nepadl žádný příhoz. Buďte první!</p>
                      ) : (
                        currentBids.map((b, idx) => (
                          <div key={b.id} className={`flex items-center justify-between p-3.5 rounded-2xl border ${
                            idx === 0 ? 'bg-blue-50/60 border-blue-200 text-blue-900' : 'bg-slate-50 border-slate-100 text-slate-700'
                          }`}>
                            <div className="flex items-center space-x-2.5">
                              <span className={`w-2.5 h-2.5 rounded-full ${idx === 0 ? 'bg-blue-600 animate-pulse' : 'bg-slate-300'}`}></span>
                              <span className="font-bold text-sm">{b.firstName}</span>
                              {idx === 0 && <span className="text-[10px] bg-blue-600 text-white px-2 py-0.5 rounded-full font-bold">Vede aukci</span>}
                            </div>
                            <div className="text-right">
                              <span className="font-black text-sm">{b.amount.toLocaleString()} Kč</span>
                              <span className="block text-[10px] text-slate-400">{b.time}</span>
                            </div>
                          </div>
                        ))
                      )}
                    </div>
                  </div>
                </div>

                <div className="lg:col-span-5 space-y-6">
                  <div className="bg-white border border-slate-200 rounded-3xl p-6 shadow-lg sticky top-20">
                    <div className="text-center pb-6 border-b border-slate-100">
                      <span className="text-xs font-semibold text-slate-400 uppercase tracking-widest">Aktuální nejvyšší nabídka</span>
                      <div className="text-4xl font-black text-blue-600 mt-2">
                        {currentAuction.currentPrice.toLocaleString()} <span className="text-lg font-bold">Kč</span>
                      </div>
                      <p className="text-xs text-slate-500 mt-1">
                        Aktuální lídr: <strong className="text-slate-800">{currentBids[0]?.firstName || 'Zatím nikdo'}</strong>
                      </p>
                    </div>

                    {currentAuction.status === 'active' ? (
                      <div className="mt-6 space-y-5">
                        <div>
                          <label className="block text-xs font-semibold text-slate-400 uppercase mb-2">Rychlé navýšení:</label>
                          <div className="grid grid-cols-4 gap-2">
                            {[100, 500, 1000, 5000].map(val => (
                              <button
                                key={val}
                                onClick={() => prepareBid(val)}
                                className="py-2.5 bg-slate-50 hover:bg-blue-50 text-slate-700 hover:text-blue-600 font-bold rounded-xl text-xs border border-slate-200 transition"
                              >
                                +{val} Kč
                              </button>
                            ))}
                          </div>
                        </div>

                        <div>
                          <div className="flex justify-between items-center text-xs font-semibold text-slate-500 mb-1">
                            <span>Posuvník navýšení</span>
                            <span className="text-blue-600 font-bold">+{bidIncrement.toLocaleString()} Kč</span>
                          </div>
                          <input 
                            type="range" 
                            min={currentAuction.minBidIncrement} 
                            max={currentAuction.minBidIncrement * 25} 
                            step={currentAuction.minBidIncrement}
                            value={bidIncrement}
                            onChange={(e) => setBidIncrement(Number(e.target.value))}
                            className="w-full accent-blue-600 cursor-pointer"
                          />
                        </div>

                        <div>
                          <label className="block text-xs font-semibold text-slate-400 uppercase mb-1">Vlastní celková částka (Kč)</label>
                          <div className="flex space-x-2">
                            <input 
                              type="number" 
                              placeholder={currentAuction.currentPrice + currentAuction.minBidIncrement}
                              value={customBidInput}
                              onChange={(e) => setCustomBidInput(e.target.value)}
                              className="flex-1 px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl text-slate-900 text-sm focus:outline-none focus:border-blue-600"
                            />
                            <button 
                              onClick={() => {
                                const val = Number(customBidInput);
                                if (!val || val <= currentAuction.currentPrice) {
                                  showToast('Částka musí být vyšší než aktuální cena.', 'error');
                                  return;
                                }
                                setTargetBidAmount(val);
                                setConfirmModal(true);
                              }}
                              className="px-4 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold rounded-xl text-xs transition border border-slate-200"
                            >
                              Ověřit
                            </button>
                          </div>
                        </div>

                        <button 
                          onClick={() => prepareBid(bidIncrement)}
                          className="w-full py-4 bg-blue-600 hover:bg-blue-700 text-white font-black rounded-2xl shadow-xl transition text-base tracking-wide flex items-center justify-center space-x-2"
                        >
                          <GavelIcon />
                          <span className="text-white">Přihodit {(currentAuction.currentPrice + bidIncrement).toLocaleString()} Kč</span>
                        </button>
                      </div>
                    ) : (
                      <div className="mt-6 text-center py-8 bg-slate-50 rounded-2xl border border-slate-200">
                        <h4 className="text-lg font-bold text-slate-800">Aukce není aktivní</h4>
                        <p className="text-xs text-slate-500 mt-1">Tato aukce skončila.</p>
                      </div>
                    )}
                  </div>
                </div>
              </main>
            </div>
          );
        }

        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    </script>
</body>
</html>