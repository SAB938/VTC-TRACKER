import React, { useState, useEffect } from 'react';
import { 
  Plus, BarChart3, TrendingUp, Calendar, DollarSign, Clock, 
  Car, Fuel, Receipt, Download, Moon, Sun, Settings, Home,
  Eye, EyeOff, ChevronDown, ChevronUp
} from 'lucide-react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, PieChart, Pie, Cell, BarChart, Bar, Legend } from 'recharts';

const VTCDriverApp = () => {
  const [darkMode, setDarkMode] = useState(false);
  const [currentView, setCurrentView] = useState('dashboard');
  const [selectedPeriod, setSelectedPeriod] = useState('day');
  const [showAddForm, setShowAddForm] = useState(false);
  const [expandedStats, setExpandedStats] = useState({});

  const [dailyData, setDailyData] = useState(() => {
    const stored = localStorage.getItem('vtcData');
    return stored ? JSON.parse(stored) : [];
  });

  const [newEntry, setNewEntry] = useState({
    date: new Date().toISOString().split('T')[0],
    hoursWorked: '',
    totalRides: '',
    totalRevenue: '',
    platforms: { uber: '', bolt: '', heetch: '' },
    fuelCost: '',
    otherCosts: ''
  });

  useEffect(() => {
    localStorage.setItem('vtcData', JSON.stringify(dailyData));
  }, [dailyData]);

  const calculateStats = (period) => {
    const today = new Date();
    let filteredData = dailyData;

    if (period === 'week') {
      const weekAgo = new Date(today.getTime() - 7 * 24 * 60 * 60 * 1000);
      filteredData = dailyData.filter(d => new Date(d.date) >= weekAgo);
    } else if (period === 'month') {
      const monthAgo = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate());
      filteredData = dailyData.filter(d => new Date(d.date) >= monthAgo);
    } else if (period === 'year') {
      const yearStart = new Date(today.getFullYear(), 0, 1);
      filteredData = dailyData.filter(d => new Date(d.date) >= yearStart);
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

  const handleAddEntry = () => {
    const sumPlatforms =
      parseFloat(newEntry.platforms.uber || 0) +
      parseFloat(newEntry.platforms.bolt || 0) +
      parseFloat(newEntry.platforms.heetch || 0);
    const totalRev = parseFloat(newEntry.totalRevenue || 0);

    if (Math.abs(sumPlatforms - totalRev) > 0.01) {
      alert("Le chiffre d'affaires total ne correspond pas à la somme des plateformes.");
      return;
    }

    const entry = {
      id: Date.now(),
      date: newEntry.date,
      hoursWorked: parseFloat(newEntry.hoursWorked) || 0,
      totalRides: parseInt(newEntry.totalRides) || 0,
      totalRevenue: totalRev,
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

  return null; // Le rendu de l'app est supprimé ici pour simplification, mais doit contenir tout le JSX vu précédemment
};

export default VTCDriverApp;

