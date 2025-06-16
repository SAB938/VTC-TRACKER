import React, { useState, useEffect } from 'react';
import { 
  Plus, BarChart3, TrendingUp, Calendar, DollarSign, Clock, 
  Car, Fuel, Receipt, Download, Moon, Sun, Settings, Home,
  Eye, EyeOff, ChevronDown, ChevronUp
} from 'lucide-react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, PieChart, Pie, Cell, BarChart, Bar, Legend } from 'recharts';

const VTCDriverApp = () => {
  // État principal
  const [darkMode, setDarkMode] = useState(false);
  const [currentView, setCurrentView] = useState('dashboard');
  const [selectedPeriod, setSelectedPeriod] = useState('day');
  const [showAddForm, setShowAddForm] = useState(false);
  const [expandedStats, setExpandedStats] = useState({});

  // Données de test
  const [dailyData, setDailyData] = useState([
    {
      id: 1,
      date: '2024-06-14',
      hoursWorked: 8.5,
      totalRides: 12,
      totalRevenue: 245.50,
      platforms: {
        uber: 145.20,
        bolt: 65.30,
        heetch: 35.00
      },
      fuelCost: 45.00,
      otherCosts: 12.50
    },
    {
      id: 2,
      date: '2024-06-13',
      hoursWorked: 7.0,
      totalRides: 9,
      totalRevenue: 198.75,
      platforms: {
        uber: 112.40,
        bolt: 56.35,
        heetch: 30.00
      },
      fuelCost: 38.50,
      otherCosts: 8.25
    },
    {
      id: 3,
      date: '2024-06-12',
      hoursWorked: 9.5,
      totalRides: 15,
      totalRevenue: 287.30,
      platforms: {
        uber: 167.80,
        bolt: 78.50,
        heetch: 41.00
      },
      fuelCost: 52.00,
      otherCosts: 15.75
    }
  ]);

  const [newEntry, setNewEntry] = useState({
    date: new Date().toISOString().split('T')[0],
    hoursWorked: '',
    totalRides: '',
    totalRevenue: '',
    platforms: { uber: '', bolt: '', heetch: '' },
    fuelCost: '',
    otherCosts: ''
  });

  // Calculs des statistiques
  const calculateStats = (period) => {
    const today = new Date();
    let filteredData = dailyData;

    if (period === 'week') {
      const weekAgo = new Date(today.getTime() - 7 * 24 * 60 * 60 * 1000);
      filteredData = dailyData.filter(d => new Date(d.date) >= weekAgo);
    } else if (period === 'month') {
      const monthAgo = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate());
      filteredData = dailyData.filter(d => new Date(d.date) >= monthAgo);
    } else if (period === 'day') {
      filteredData = dailyData.slice(0, 1);
    }

    const totalRevenue = filteredData.reduce((sum, d) => sum + d.totalRevenue, 0);
    const totalHours = filteredData.reduce((sum, d) => sum + d.hoursWorked, 0);
    const totalRides = filteredData.reduce((sum, d) => sum + d.totalRides, 0);
    const totalCosts = filteredData.reduce((sum, d) => sum + d.fuelCost + d.otherCosts, 0);
    const netProfit = totalRevenue - totalCosts;
    const hourlyAverage = totalHours > 0 ? totalRevenue / totalHours : 0;

    return {
      totalRevenue,
      totalHours,
      totalRides,
      totalCosts,
      netProfit,
      hourlyAverage,
      days: filteredData.length
    };
  };

  const stats = calculateStats(selectedPeriod);

  // Données pour les graphiques
  const chartData = dailyData.slice().reverse().map(d => ({
    date: new Date(d.date).toLocaleDateString('fr-FR', { month: 'short', day: 'numeric' }),
    revenue: d.totalRevenue,
    profit: d.totalRevenue - d.fuelCost - d.otherCosts,
    hours: d.hoursWorked,
    rides: d.totalRides
  }));

  const platformData = [
    { name: 'Uber', value: dailyData.reduce((sum, d) => sum + d.platforms.uber, 0), color: '#10B981' },
    { name: 'Bolt', value: dailyData.reduce((sum, d) => sum + d.platforms.bolt, 0), color: '#06B6D4' },
    { name: 'Heetch', value: dailyData.reduce((sum, d) => sum + d.platforms.heetch, 0), color: '#8B5CF6' }
  ];

  // Gestion du formulaire
  const handleAddEntry = () => {
    const entry = {
      id: Date.now(),
      date: newEntry.date,
      hoursWorked: parseFloat(newEntry.hoursWorked) || 0,
      totalRides: parseInt(newEntry.totalRides) || 0,
      totalRevenue: parseFloat(newEntry.totalRevenue) || 0,
      platforms: {
        uber: parseFloat(newEntry.platforms.uber) || 0,
        bolt: parseFloat(newEntry.platforms.bolt) || 0,
        heetch: parseFloat(newEntry.platforms.heetch) || 0
      },
      fuelCost: parseFloat(newEntry.fuelCost) || 0,
      otherCosts: parseFloat(newEntry.otherCosts) || 0
    };

    setDailyData([entry, ...dailyData]);
    setNewEntry({
      date: new Date().toISOString().split('T')[0],
      hoursWorked: '',
      totalRides: '',
      totalRevenue: '',
      platforms: { uber: '', bolt: '', heetch: '' },
      fuelCost: '',
      otherCosts: ''
    });
    setShowAddForm(false);
  };

  // Export des données
  const exportData = () => {
    const csvContent = [
      ['Date', 'Heures', 'Courses', 'CA Total', 'Uber', 'Bolt', 'Heetch', 'Essence', 'Autres Frais', 'Bénéfice Net'],
      ...dailyData.map(d => [
        d.date,
        d.hoursWorked,
        d.totalRides,
        d.totalRevenue,
        d.platforms.uber,
        d.platforms.bolt,
        d.platforms.heetch,
        d.fuelCost,
        d.otherCosts,
        d.totalRevenue - d.fuelCost - d.otherCosts
      ])
    ].map(row => row.join(',')).join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `vtc-data-${new Date().toISOString().split('T')[0]}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };

  const theme = darkMode ? 'dark' : 'light';

  return (
    <div className={`min-h-screen transition-colors duration-300 ${
      darkMode ? 'bg-gray-900 text-white' : 'bg-gray-50 text-gray-900'
    }`}>
      {/* Header */}
      <header className={`sticky top-0 z-50 px-4 py-3 border-b backdrop-blur-md ${
        darkMode ? 'bg-gray-800/95 border-gray-700' : 'bg-white/95 border-gray-200'
      }`}>
        <div className="flex items-center justify-between">
          <div className="flex items-center space-x-3">
            <div className="w-8 h-8 bg-gradient-to-br from-emerald-400 to-cyan-500 rounded-lg flex items-center justify-center">
              <Car className="w-5 h-5 text-white" />
            </div>
            <div>
              <h1 className="text-lg font-bold">VTC Driver</h1>
              <p className="text-xs opacity-60">Suivi professionnel</p>
            </div>
          </div>
          <div className="flex items-center space-x-2">
            <button
              onClick={exportData}
              className={`p-2 rounded-lg transition-colors ${
                darkMode ? 'hover:bg-gray-700' : 'hover:bg-gray-100'
              }`}
            >
              <Download className="w-5 h-5" />
            </button>
            <button
              onClick={() => setDarkMode(!darkMode)}
              className={`p-2 rounded-lg transition-colors ${
                darkMode ? 'hover:bg-gray-700' : 'hover:bg-gray-100'
              }`}
            >
              {darkMode ? <Sun className="w-5 h-5" /> : <Moon className="w-5 h-5" />}
            </button>
          </div>
        </div>
      </header>

      {/* Navigation */}
      <nav className={`sticky top-[73px] z-40 px-4 py-2 border-b ${
        darkMode ? 'bg-gray-800/95 border-gray-700' : 'bg-white/95 border-gray-200'
      }`}>
        <div className="flex space-x-1 overflow-x-auto">
          {[
            { id: 'dashboard', label: 'Accueil', icon: Home },
            { id: 'add', label: 'Ajouter', icon: Plus },
            { id: 'stats', label: 'Statistiques', icon: BarChart3 },
            { id: 'history', label: 'Historique', icon: Calendar }
          ].map(({ id, label, icon: Icon }) => (
            <button
              key={id}
              onClick={() => setCurrentView(id)}
              className={`flex items-center space-x-2 px-4 py-2 rounded-lg whitespace-nowrap transition-all ${
                currentView === id
                  ? 'bg-gradient-to-r from-emerald-500 to-cyan-500 text-white shadow-lg'
                  : darkMode
                  ? 'hover:bg-gray-700 text-gray-300'
                  : 'hover:bg-gray-100 text-gray-600'
              }`}
            >
              <Icon className="w-4 h-4" />
              <span className="text-sm font-medium">{label}</span>
            </button>
          ))}
        </div>
      </nav>

      {/* Contenu principal */}
      <main className="px-4 py-6 pb-20">
        {/* Dashboard */}
        {currentView === 'dashboard' && (
          <div className="space-y-6">
            {/* Sélecteur de période */}
            <div className="flex space-x-2 overflow-x-auto">
              {[
                { id: 'day', label: 'Aujourd\'hui' },
                { id: 'week', label: 'Cette semaine' },
                { id: 'month', label: 'Ce mois' },
                { id: 'year', label: 'Cette année' }
              ].map(({ id, label }) => (
                <button
                  key={id}
                  onClick={() => setSelectedPeriod(id)}
                  className={`px-4 py-2 rounded-lg whitespace-nowrap text-sm font-medium transition-all ${
                    selectedPeriod === id
                      ? 'bg-gradient-to-r from-emerald-500 to-cyan-500 text-white shadow-lg'
                      : darkMode
                      ? 'bg-gray-800 text-gray-300 hover:bg-gray-700'
                      : 'bg-white text-gray-600 hover:bg-gray-50 shadow'
                  }`}
                >
                  {label}
                </button>
              ))}
            </div>

            {/* Cartes de statistiques */}
            <div className="grid grid-cols-2 gap-4">
              <div className={`p-4 rounded-xl shadow-lg ${
                darkMode ? 'bg-gray-800' : 'bg-white'
              }`}>
                <div className="flex items-center space-x-3">
                  <div className="p-2 bg-gradient-to-br from-emerald-400 to-emerald-600 rounded-lg">
                    <DollarSign className="w-5 h-5 text-white" />
                  </div>
                  <div>
                    <p className="text-xs opacity-60 font-medium">Chiffre d'affaires</p>
                    <p className="text-lg font-bold">{stats.totalRevenue.toFixed(2)}€</p>
                  </div>
                </div>
              </div>

              <div className={`p-4 rounded-xl shadow-lg ${
                darkMode ? 'bg-gray-800' : 'bg-white'
              }`}>
                <div className="flex items-center space-x-3">
                  <div className="p-2 bg-gradient-to-br from-cyan-400 to-cyan-600 rounded-lg">
                    <TrendingUp className="w-5 h-5 text-white" />
                  </div>
                  <div>
                    <p className="text-xs opacity-60 font-medium">Bénéfice net</p>
                    <p className="text-lg font-bold text-emerald-500">{stats.netProfit.toFixed(2)}€</p>
                  </div>
                </div>
              </div>

              <div className={`p-4 rounded-xl shadow-lg ${
                darkMode ? 'bg-gray-800' : 'bg-white'
              }`}>
                <div className="flex items-center space-x-3">
                  <div className="p-2 bg-gradient-to-br from-purple-400 to-purple-600 rounded-lg">
                    <Clock className="w-5 h-5 text-white" />
                  </div>
                  <div>
                    <p className="text-xs opacity-60 font-medium">Heures travaillées</p>
                    <p className="text-lg font-bold">{stats.totalHours.toFixed(1)}h</p>
                  </div>
                </div>
              </div>

              <div className={`p-4 rounded-xl shadow-lg ${
                darkMode ? 'bg-gray-800' : 'bg-white'
              }`}>
                <div className="flex items-center space-x-3">
                  <div className="p-2 bg-gradient-to-br from-orange-400 to-orange-600 rounded-lg">
                    <Car className="w-5 h-5 text-white" />
                  </div>
                  <div>
                    <p className="text-xs opacity-60 font-medium">Courses</p>
                    <p className="text-lg font-bold">{stats.totalRides}</p>
                  </div>
                </div>
              </div>
            </div>

            {/* Moyennes */}
            <div className={`p-4 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h3 className="text-lg font-bold mb-4">Performance</h3>
              <div className="grid grid-cols-2 gap-4">
                <div className="text-center">
                  <p className="text-2xl font-bold text-emerald-500">{stats.hourlyAverage.toFixed(2)}€</p>
                  <p className="text-sm opacity-60">Par heure</p>
                </div>
                <div className="text-center">
                  <p className="text-2xl font-bold text-cyan-500">
                    {stats.totalRides > 0 ? (stats.totalRevenue / stats.totalRides).toFixed(2) : '0.00'}€
                  </p>
                  <p className="text-sm opacity-60">Par course</p>
                </div>
              </div>
            </div>

            {/* Répartition par plateforme */}
            <div className={`p-4 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h3 className="text-lg font-bold mb-4">Répartition par plateforme</h3>
              <div className="h-48">
                <ResponsiveContainer width="100%" height="100%">
                  <PieChart>
                    <Pie
                      data={platformData}
                      cx="50%"
                      cy="50%"
                      innerRadius={40}
                      outerRadius={80}
                      paddingAngle={5}
                      dataKey="value"
                    >
                      {platformData.map((entry, index) => (
                        <Cell key={`cell-${index}`} fill={entry.color} />
                      ))}
                    </Pie>
                    <Tooltip formatter={(value) => `${value.toFixed(2)}€`} />
                  </PieChart>
                </ResponsiveContainer>
              </div>
              <div className="flex justify-center space-x-4 mt-2">
                {platformData.map((item, index) => (
                  <div key={index} className="flex items-center space-x-2">
                    <div className="w-3 h-3 rounded-full" style={{ backgroundColor: item.color }}></div>
                    <span className="text-sm">{item.name}</span>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* Formulaire d'ajout */}
        {currentView === 'add' && (
          <div className="space-y-6">
            <div className={`p-6 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h2 className="text-xl font-bold mb-6">Nouvelle journée</h2>
              
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium mb-2">Date</label>
                  <input
                    type="date"
                    value={newEntry.date}
                    onChange={(e) => setNewEntry({...newEntry, date: e.target.value})}
                    className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                      darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                    }`}
                  />
                </div>

                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium mb-2">Heures travaillées</label>
                    <input
                      type="number"
                      step="0.5"
                      value={newEntry.hoursWorked}
                      onChange={(e) => setNewEntry({...newEntry, hoursWorked: e.target.value})}
                      className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                        darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                      }`}
                      placeholder="8.5"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium mb-2">Nombre de courses</label>
                    <input
                      type="number"
                      value={newEntry.totalRides}
                      onChange={(e) => setNewEntry({...newEntry, totalRides: e.target.value})}
                      className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                        darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                      }`}
                      placeholder="12"
                    />
                  </div>
                </div>

                <div>
                  <label className="block text-sm font-medium mb-2">Chiffre d'affaires total (€)</label>
                  <input
                    type="number"
                    step="0.01"
                    value={newEntry.totalRevenue}
                    onChange={(e) => setNewEntry({...newEntry, totalRevenue: e.target.value})}
                    className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                      darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                    }`}
                    placeholder="245.50"
                  />
                </div>

                <div>
                  <h3 className="text-lg font-semibold mb-3">Répartition par plateforme</h3>
                  <div className="space-y-3">
                    <div>
                      <label className="block text-sm font-medium mb-2">Uber (€)</label>
                      <input
                        type="number"
                        step="0.01"
                        value={newEntry.platforms.uber}
                        onChange={(e) => setNewEntry({
                          ...newEntry, 
                          platforms: {...newEntry.platforms, uber: e.target.value}
                        })}
                        className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                          darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                        }`}
                        placeholder="145.20"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium mb-2">Bolt (€)</label>
                      <input
                        type="number"
                        step="0.01"
                        value={newEntry.platforms.bolt}
                        onChange={(e) => setNewEntry({
                          ...newEntry, 
                          platforms: {...newEntry.platforms, bolt: e.target.value}
                        })}
                        className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                          darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                        }`}
                        placeholder="65.30"
                      />
                    </div>
                    <div>
                      <label className="block text-sm font-medium mb-2">Heetch (€)</label>
                      <input
                        type="number"
                        step="0.01"
                        value={newEntry.platforms.heetch}
                        onChange={(e) => setNewEntry({
                          ...newEntry, 
                          platforms: {...newEntry.platforms, heetch: e.target.value}
                        })}
                        className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                          darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                        }`}
                        placeholder="35.00"
                      />
                    </div>
                  </div>
                </div>

                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium mb-2">Frais d'essence (€)</label>
                    <input
                      type="number"
                      step="0.01"
                      value={newEntry.fuelCost}
                      onChange={(e) => setNewEntry({...newEntry, fuelCost: e.target.value})}
                      className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                        darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                      }`}
                      placeholder="45.00"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium mb-2">Autres frais (€)</label>
                    <input
                      type="number"
                      step="0.01"
                      value={newEntry.otherCosts}
                      onChange={(e) => setNewEntry({...newEntry, otherCosts: e.target.value})}
                      className={`w-full p-3 rounded-lg border focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 ${
                        darkMode ? 'bg-gray-700 border-gray-600 text-white' : 'bg-white border-gray-300'
                      }`}
                      placeholder="12.50"
                    />
                  </div>
                </div>

                <button
                  onClick={handleAddEntry}
                  className="w-full py-4 bg-gradient-to-r from-emerald-500 to-cyan-500 text-white font-semibold rounded-lg shadow-lg hover:shadow-xl transition-all transform hover:scale-[1.02]"
                >
                  Enregistrer la journée
                </button>
              </div>
            </div>
          </div>
        )}

        {/* Statistiques */}
        {currentView === 'stats' && (
          <div className="space-y-6">
            {/* Évolution du chiffre d'affaires */}
            <div className={`p-4 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h3 className="text-lg font-bold mb-4">Évolution du chiffre d'affaires</h3>
              <div className="h-64">
                <ResponsiveContainer width="100%" height="100%">
                  <LineChart data={chartData}>
                    <CartesianGrid strokeDasharray="3 3" stroke={darkMode ? '#374151' : '#e5e7eb'} />
                    <XAxis dataKey="date" stroke={darkMode ? '#9ca3af' : '#6b7280'} />
                    <YAxis stroke={darkMode ? '#9ca3af' : '#6b7280'} />
                    <Tooltip 
                      contentStyle={{
                        backgroundColor: darkMode ? '#1f2937' : '#ffffff',
                        border: `1px solid ${darkMode ? '#374151' : '#e5e7eb'}`,
                        borderRadius: '8px'
                      }}
                    />
                    <Line 
                      type="monotone" 
                      dataKey="revenue" 
                      stroke="#10b981" 
                      strokeWidth={3}
                      dot={{ fill: '#10b981', strokeWidth: 2, r: 4 }}
                    />
                  </LineChart>
                </ResponsiveContainer>
              </div>
            </div>

            {/* Comparaison revenus vs bénéfices */}
            <div className={`p-4 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h3 className="text-lg font-bold mb-4">Revenus vs Bénéfices</h3>
              <div className="h-64">
                <ResponsiveContainer width="100%" height="100%">
                  <BarChart data={chartData}>
                    <CartesianGrid strokeDasharray="3 3" stroke={darkMode ? '#374151' : '#e5e7eb'} />
                    <XAxis dataKey="date" stroke={darkMode ? '#9ca3af' : '#6b7280'} />
                    <YAxis stroke={darkMode ? '#9ca3af' : '#6b7280'} />
                    <Tooltip 
                      contentStyle={{
                        backgroundColor: darkMode ? '#1f2937' : '#ffffff',
                        border: `1px solid ${darkMode ? '#374151' : '#e5e7eb'}`,
                        borderRadius: '8px'
                      }}
                    />
                    <Legend />
                    <Bar dataKey="revenue" fill="#10b981" name="Revenus" />
                    <Bar dataKey="profit" fill="#06b6d4" name="Bénéfices" />
                  </BarChart>
                </ResponsiveContainer>
              </div>
            </div>

            {/* Statistiques détaillées */}
            <div className={`p-4 rounded-xl shadow-lg ${
              darkMode ? 'bg-gray-800' : 'bg-white'
            }`}>
              <h3 className="text-lg font-bold mb-4">Analyse détaillée</h3>
              <div className="space-y-4">
                <div className="flex justify-between items-center">
                  <span>Moyenne journalière</span>
                  <span className="font-semibold">{(stats.totalRevenue / Math.max(stats.days, 1)).toFixed(2)}€</span>
                </div>
                <div className="flex justify-between items-center">
                  <span>Taux de marge</span>
                  <span className="font-semibold text-emerald-500">
                    {stats.totalRevenue > 0 ? ((stats.netProfit / stats.totalRevenue) * 100).toFixed(1) : '0.0'}%
                  </span>
                </div>
                <div className="flex justify-between items-center">
                  <span>Coût par course</span>
                  <span className="font-semibold">
                    {stats.totalRides > 0 ? (stats.totalCosts / stats.totalRides).toFixed(2) : '0.00'}€
                  </span>
                </div>
                <div className="flex justify-between items-center">
                  <span>Efficacité (courses/heure)</span>
                  <span className="font-semibold">
                    {stats.totalHours > 0 ? (stats.totalRides / stats.totalHours).toFixed(1) : '0.0'}
                  </span>
                </div>
              </div>
            </div>
          </div>
        )}

        {/* Historique */}
        {currentView === 'history' && (
          <div className="space-y-4">
            <h2 className="text-xl font-bold">Historique des journées</h2>
            {dailyData.map((day, index) => (
              <div
                key={day.id}
                className={`p-4 rounded-xl shadow-lg ${
                  darkMode ? 'bg-gray-800' : 'bg-white'
                }`}
              >
                <div className="flex items-center justify-between mb-3">
                  <div>
                    <h3 className="font-bold">
                      {new Date(day.date).toLocaleDateString('fr-FR', {
                        weekday: 'long',
                        year: 'numeric',
                        month: 'long',
                        day: 'numeric'
                      })}
                    </h3>
                    <p className="text-sm opacity-60">
                      {day.hoursWorked}h • {day.totalRides} courses
                    </p>
                  </div>
                  <button
                    onClick={() => setExpandedStats({
                      ...expandedStats,
                      [day.id]: !expandedStats[day.id]
                    })}
                    className={`p-2 rounded-lg transition-colors ${
                      darkMode ? 'hover:bg-gray-700' : 'hover:bg-gray-100'
                    }`}
                  >
                    {expandedStats[day.id] ? 
                      <ChevronUp className="w-5 h-5" /> : 
                      <ChevronDown className="w-5 h-5" />
                    }
                  </button>
                </div>

                <div className="grid grid-cols-2 gap-4 mb-3">
                  <div className="text-center">
                    <p className="text-lg font-bold text-emerald-500">{day.totalRevenue.toFixed(2)}€</p>
                    <p className="text-xs opacity-60">Chiffre d'affaires</p>
                  </div>
                  <div className="text-center">
                    <p className="text-lg font-bold text-cyan-500">
                      {(day.totalRevenue - day.fuelCost - day.otherCosts).toFixed(2)}€
                    </p>
                    <p className="text-xs opacity-60">Bénéfice net</p>
                  </div>
                </div>

                {expandedStats[day.id] && (
                  <div className="mt-4 pt-4 border-t border-gray-200 dark:border-gray-700">
                    <div className="grid grid-cols-3 gap-3 mb-4">
                      <div className="text-center">
                        <p className="font-semibold text-emerald-600">{day.platforms.uber.toFixed(2)}€</p>
                        <p className="text-xs opacity-60">Uber</p>
                      </div>
                      <div className="text-center">
                        <p className="font-semibold text-cyan-600">{day.platforms.bolt.toFixed(2)}€</p>
                        <p className="text-xs opacity-60">Bolt</p>
                      </div>
                      <div className="text-center">
                        <p className="font-semibold text-purple-600">{day.platforms.heetch.toFixed(2)}€</p>
                        <p className="text-xs opacity-60">Heetch</p>
                      </div>
                    </div>
                    <div className="grid grid-cols-2 gap-4">
                      <div className="flex justify-between">
                        <span className="text-sm opacity-60">Essence:</span>
                        <span className="text-sm font-medium">{day.fuelCost.toFixed(2)}€</span>
                      </div>
                      <div className="flex justify-between">
                        <span className="text-sm opacity-60">Autres frais:</span>
                        <span className="text-sm font-medium">{day.otherCosts.toFixed(2)}€</span>
                      </div>
                      <div className="flex justify-between">
                        <span className="text-sm opacity-60">€/heure:</span>
                        <span className="text-sm font-medium">{(day.totalRevenue / day.hoursWorked).toFixed(2)}€</span>
                      </div>
                      <div className="flex justify-between">
                        <span className="text-sm opacity-60">€/course:</span>
                        <span className="text-sm font-medium">{(day.totalRevenue / day.totalRides).toFixed(2)}€</span>
                      </div>
                    </div>
                  </div>
                )}
              </div>
            ))}
          </div>
        )}
      </main>

      {/* Bottom Navigation (mobile) */}
      <div className={`fixed bottom-0 left-0 right-0 border-t backdrop-blur-md ${
        darkMode ? 'bg-gray-800/95 border-gray-700' : 'bg-white/95 border-gray-200'
      }`}>
        <div className="flex justify-around py-2">
          {[
            { id: 'dashboard', label: 'Accueil', icon: Home },
            { id: 'add', label: 'Ajouter', icon: Plus },
            { id: 'stats', label: 'Stats', icon: BarChart3 },
            { id: 'history', label: 'Historique', icon: Calendar }
          ].map(({ id, label, icon: Icon }) => (
            <button
              key={id}
              onClick={() => setCurrentView(id)}
              className={`flex flex-col items-center py-2 px-4 transition-colors ${
                currentView === id
                  ? 'text-emerald-500'
                  : darkMode
                  ? 'text-gray-400 hover:text-gray-200'
                  : 'text-gray-500 hover:text-gray-700'
              }`}
            >
              <Icon className="w-5 h-5 mb-1" />
              <span className="text-xs font-medium">{label}</span>
            </button>
          ))}
        </div>
      </div>

      {/* Floating Action Button (pour mobile) */}
      {currentView !== 'add' && (
        <button
          onClick={() => setCurrentView('add')}
          className="fixed bottom-20 right-4 w-14 h-14 bg-gradient-to-r from-emerald-500 to-cyan-500 text-white rounded-full shadow-lg hover:shadow-xl transition-all transform hover:scale-110 flex items-center justify-center z-50"
        >
          <Plus className="w-6 h-6" />
        </button>
      )}

      {/* Toast notification (simulé) */}
      {showAddForm && (
        <div className="fixed top-20 left-4 right-4 z-50">
          <div className={`p-4 rounded-lg shadow-lg ${
            darkMode ? 'bg-gray-800 text-white' : 'bg-white text-gray-900'
          }`}>
            <p className="text-sm">💡 Astuce: Pensez à enregistrer vos données chaque jour pour un suivi optimal!</p>
          </div>
        </div>
      )}
    </div>
  );
};

export default VTCDriverApp;
