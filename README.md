import React, { useState, useMemo } from 'react';
import {
  Wallet,
  TrendingUp,
  TrendingDown,
  Plus,
  Trash2,
  Calendar,
  Building2,
  CreditCard,
  PieChart,
  BarChart3,
  Utensils,
  Wrench,
  Receipt,
  Briefcase,
  MoreHorizontal,
  ArrowUpRight,
  ArrowDownRight,
  Download,
  Printer,
  Filter,
  Search,
  X,
  ChevronRight,
  ChevronLeft,
  Home
} from 'lucide-react';
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  PieChart as RePieChart,
  Pie,
  Cell,
  LineChart,
  Line,
  Legend
} from 'recharts';

interface Transaction {
  id: string;
  date: string;
  amount: number;
  type: 'deposit' | 'expense';
  bankAccount: string;
  description: string;
  category?: string;
}

interface BankAccount {
  id: string;
  name: string;
  owner: string;
  balance: number;
  logo: string;
  color: string;
}

const expenseCategories = [
  { id: 'lunch', name: 'ناهار و عصرانه', icon: Utensils, color: '#f97316' },
  { id: 'spare_parts', name: 'لوازم یدکی', icon: Wrench, color: '#3b82f6' },
  { id: 'bills', name: 'قبوض', icon: Receipt, color: '#ef4444' },
  { id: 'services', name: 'خدمات', icon: Briefcase, color: '#8b5cf6' },
  { id: 'misc', name: 'متفرقه', icon: MoreHorizontal, color: '#6b7280' }
];

const bankAccounts: BankAccount[] = [
  {
    id: 'mellat_shahin',
    name: 'بانک ملی',
    owner: 'شاهین شبستری',
    balance: 0,
    logo: '🏛️',
    color: '#dc2626'
  },
  {
    id: 'saman_shahin',
    name: 'بانک سامان',
    owner: 'شاهین شبستری',
    balance: 0,
    logo: '🏦',
    color: '#2563eb'
  },
  {
    id: 'mellat_elahah',
    name: 'بانک ملی',
    owner: 'الهه هادی دولابی فرد',
    balance: 0,
    logo: '🏛️',
    color: '#dc2626'
  }
];

const months = [
  'فروردین', 'اردیبهشت', 'خرداد', 'تیر', 'مرداد', 'شهریور',
  'مهر', 'آبان', 'آذر', 'دی', 'بهمن', 'اسفند'
];

const formatToman = (amount: number) => {
  return new Intl.NumberFormat('fa-IR').format(Math.round(amount / 10)) + ' تومان';
};

const formatNumber = (num: number) => {
  return new Intl.NumberFormat('fa-IR').format(num);
};

const toPersianNumber = (num: string | number) => {
  const persianDigits = ['۰', '۱', '۲', '۳', '۴', '۵', '۶', '۷', '۸', '۹'];
  return num.toString().replace(/\d/g, (w) => persianDigits[+w]);
};

const getDayName = (dateStr: string) => {
  const days = ['یکشنبه', 'دوشنبه', 'سه‌شنبه', 'چهارشنبه', 'پنجشنبه', 'جمعه', 'شنبه'];
  const date = new Date(dateStr);
  return days[date.getDay()];
};

const getMonthFromDate = (dateStr: string) => {
  const parts = dateStr.split('/');
  if (parts.length >= 2) {
    return parseInt(parts[1]) - 1;
  }
  return new Date().getMonth();
};

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const tooltipFormatter = (value: any) => {
  if (typeof value === 'number') {
    return [formatNumber(value) + ' تومان', ''];
  }
  return ['', ''];
};

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const pieTooltipFormatter = (value: any) => {
  if (typeof value === 'number') {
    return formatToman(value);
  }
  return '';
};

export default function App() {
  const [transactions, setTransactions] = useState<Transaction[]>([
    { id: '1', date: '1403/01/01', amount: 50000000, type: 'deposit', bankAccount: 'mellat_shahin', description: 'واریز اولیه', category: '' },
    { id: '2', date: '1403/01/02', amount: 2500000, type: 'expense', bankAccount: 'mellat_shahin', description: 'ناهار کارگران', category: 'lunch' },
    { id: '3', date: '1403/01/03', amount: 15000000, type: 'expense', bankAccount: 'saman_shahin', description: 'خرید لوازم یدکی', category: 'spare_parts' },
    { id: '4', date: '1403/01/04', amount: 800000, type: 'expense', bankAccount: 'mellat_elahah', description: 'قبض برق', category: 'bills' },
  ]);
  const [showForm, setShowForm] = useState(false);
  const [activeView, setActiveView] = useState<'dashboard' | 'monthly'>('dashboard');
  const [selectedMonth, setSelectedMonth] = useState<number>(new Date().getMonth());
  const [currentYear, setCurrentYear] = useState<number>(1403);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterType, setFilterType] = useState<'all' | 'deposit' | 'expense'>('all');
  const [formData, setFormData] = useState({
    date: new Date().toISOString().split('T')[0],
    amount: '',
    type: 'expense' as 'deposit' | 'expense',
    bankAccount: 'mellat_shahin',
    description: '',
    category: 'misc'
  });

  const stats = useMemo(() => {
    const totalDeposits = transactions
      .filter(t => t.type === 'deposit')
      .reduce((sum, t) => sum + t.amount, 0);
    const totalExpenses = transactions
      .filter(t => t.type === 'expense')
      .reduce((sum, t) => sum + t.amount, 0);
    const balance = totalDeposits - totalExpenses;

    const bankBalances = bankAccounts.map(account => {
      const accountDeposits = transactions
        .filter(t => t.type === 'deposit' && t.bankAccount === account.id)
        .reduce((sum, t) => sum + t.amount, 0);
      const accountExpenses = transactions
        .filter(t => t.type === 'expense' && t.bankAccount === account.id)
        .reduce((sum, t) => sum + t.amount, 0);
      return {
        ...account,
        balance: accountDeposits - accountExpenses
      };
    });

    return { totalDeposits, totalExpenses, balance, bankBalances };
  }, [transactions]);

  const categoryStats = useMemo(() => {
    const categoryData: { [key: string]: number } = {};
    expenseCategories.forEach(cat => categoryData[cat.id] = 0);
    
    transactions
      .filter(t => t.type === 'expense')
      .forEach(t => {
        if (t.category) {
          categoryData[t.category] = (categoryData[t.category] || 0) + t.amount;
        }
      });

    return expenseCategories.map(cat => ({
      name: cat.name,
      value: categoryData[cat.id] || 0,
      color: cat.color
    })).filter(item => item.value > 0);
  }, [transactions]);

  const weeklyData = useMemo(() => {
    const last7Days: { [key: string]: { deposits: number; expenses: number } } = {};
    const today = new Date();
    
    for (let i = 6; i >= 0; i--) {
      const d = new Date(today);
      d.setDate(d.getDate() - i);
      const dateStr = d.toISOString().split('T')[0];
      last7Days[dateStr] = { deposits: 0, expenses: 0 };
    }

    transactions.forEach(t => {
      if (last7Days[t.date]) {
        if (t.type === 'deposit') {
          last7Days[t.date].deposits += t.amount;
        } else {
          last7Days[t.date].expenses += t.amount;
        }
      }
    });

    return Object.entries(last7Days).map(([date, data]) => ({
      date: getDayName(date),
      deposits: Math.round(data.deposits / 10),
      expenses: Math.round(data.expenses / 10)
    }));
  }, [transactions]);

  const categoryWeeklyData = useMemo(() => {
    const last7Days: { [key: string]: { [category: string]: number } } = {};
    const today = new Date();
    
    for (let i = 6; i >= 0; i--) {
      const d = new Date(today);
      d.setDate(d.getDate() - i);
      const dateStr = d.toISOString().split('T')[0];
      last7Days[dateStr] = {};
    }

    transactions
      .filter(t => t.type === 'expense')
      .forEach(t => {
        if (last7Days[t.date] && t.category) {
          last7Days[t.date][t.category] = (last7Days[t.date][t.category] || 0) + t.amount;
        }
      });

    return Object.entries(last7Days).map(([date, data]) => ({
      date: getDayName(date),
      ...data
    }));
  }, [transactions]);

  const monthlyData = useMemo(() => {
    const monthData: { [key: number]: { deposits: number; expenses: number } } = {};
    
    months.forEach((_, idx) => {
      monthData[idx] = { deposits: 0, expenses: 0 };
    });

    transactions.forEach(t => {
      const month = getMonthFromDate(t.date);
      if (monthData[month]) {
        if (t.type === 'deposit') {
          monthData[month].deposits += t.amount;
        } else {
          monthData[month].expenses += t.amount;
        }
      }
    });

    return months.map((month, idx) => ({
      name: month,
      deposits: Math.round(monthData[idx].deposits / 10),
      expenses: Math.round(monthData[idx].expenses / 10),
      month: idx
    }));
  }, [transactions]);

  const selectedMonthData = useMemo(() => {
    const filteredTransactions = transactions.filter(t => {
      const tMonth = getMonthFromDate(t.date);
      return tMonth === selectedMonth;
    });

    const monthDeposits = filteredTransactions
      .filter(t => t.type === 'deposit')
      .reduce((sum, t) => sum + t.amount, 0);
    const monthExpenses = filteredTransactions
      .filter(t => t.type === 'expense')
      .reduce((sum, t) => sum + t.amount, 0);

    const monthCategoryData: { [key: string]: number } = {};
    filteredTransactions
      .filter(t => t.type === 'expense')
      .forEach(t => {
        if (t.category) {
          monthCategoryData[t.category] = (monthCategoryData[t.category] || 0) + t.amount;
        }
      });

    return {
      deposits: monthDeposits,
      expenses: monthExpenses,
      transactions: filteredTransactions,
      categoryData: expenseCategories.map(cat => ({
        name: cat.name,
        value: monthCategoryData[cat.id] || 0,
        color: cat.color
      })).filter(item => item.value > 0)
    };
  }, [transactions, selectedMonth]);

  const filteredTransactions = useMemo(() => {
    return transactions.filter(t => {
      const matchesSearch = t.description.toLowerCase().includes(searchTerm.toLowerCase()) ||
                          formatToman(t.amount).includes(searchTerm);
      const matchesType = filterType === 'all' || t.type === filterType;
      return matchesSearch && matchesType;
    }).sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());
  }, [transactions, searchTerm, filterType]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const newTransaction: Transaction = {
      id: Date.now().toString(),
      date: formData.date,
      amount: parseFloat(formData.amount) * 10, // Convert toman to rial for storage
      type: formData.type,
      bankAccount: formData.bankAccount,
      description: formData.description,
      category: formData.type === 'expense' ? formData.category : ''
    };
    setTransactions([newTransaction, ...transactions]);
    setShowForm(false);
    setFormData({
      date: new Date().toISOString().split('T')[0],
      amount: '',
      type: 'expense',
      bankAccount: 'mellat_shahin',
      description: '',
      category: 'misc'
    });
  };

  const deleteTransaction = (id: string) => {
    setTransactions(transactions.filter(t => t.id !== id));
  };

  const handlePrint = () => {
    window.print();
  };

  const exportToCSV = () => {
    const headers = ['تاریخ', 'نوع', 'مبلغ (تومان)', 'حساب', 'دسته‌بندی', 'شرح'];
    const rows = transactions.map(t => [
      toPersianNumber(t.date),
      t.type === 'deposit' ? 'واریزی' : 'هزینه',
      toPersianNumber(Math.round(t.amount / 10)),
      bankAccounts.find(b => b.id === t.bankAccount)?.name + ' - ' + 
      bankAccounts.find(b => b.id === t.bankAccount)?.owner,
      expenseCategories.find(c => c.id === t.category)?.name || '-',
      t.description
    ]);
    
    const csvContent = [headers, ...rows]
      .map(row => row.join(','))
      .join('\n');
    
    const blob = new Blob(['\ufeff' + csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = `گزارش-تنخواه-${currentYear}.csv`;
    link.click();
  };

  return (
    <div dir="rtl" className="min-h-screen bg-gradient-to-br from-slate-50 to-slate-100 text-slate-800">
      {/* Header */}
      <header className="bg-gradient-to-r from-blue-600 via-blue-700 to-blue-800 text-white shadow-xl print:hidden">
        <div className="max-w-7xl mx-auto px-4 py-4">
          <div className="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
            <div className="flex items-center gap-3">
              <div className="bg-white/20 p-3 rounded-xl backdrop-blur-sm">
                <Building2 className="w-8 h-8" />
              </div>
              <div>
                <h1 className="text-2xl font-bold">سیستم مدیریت تنخواه</h1>
                <p className="text-blue-100 text-sm">کارخانه تولیدی</p>
              </div>
            </div>
            
            <div className="flex flex-wrap items-center gap-2">
              <button
                onClick={() => setActiveView('dashboard')}
                className={`flex items-center gap-2 px-4 py-2 rounded-lg transition-all ${
                  activeView === 'dashboard' 
                    ? 'bg-white text-blue-700 shadow-lg' 
                    : 'bg-white/20 hover:bg-white/30'
                }`}
              >
                <Home className="w-5 h-5" />
                داشبورد
              </button>
              <button
                onClick={() => setActiveView('monthly')}
                className={`flex items-center gap-2 px-4 py-2 rounded-lg transition-all ${
                  activeView === 'monthly' 
                    ? 'bg-white text-blue-700 shadow-lg' 
                    : 'bg-white/20 hover:bg-white/30'
                }`}
              >
                <Calendar className="w-5 h-5" />
                نمای سالانه
              </button>
              <button
                onClick={() => setShowForm(true)}
                className="flex items-center gap-2 bg-emerald-500 hover:bg-emerald-600 px-4 py-2 rounded-lg shadow-lg transition-all"
              >
                <Plus className="w-5 h-5" />
                ثبت تراکنش
              </button>
              <button
                onClick={handlePrint}
                className="flex items-center gap-2 bg-white/20 hover:bg-white/30 px-3 py-2 rounded-lg transition-all"
              >
                <Printer className="w-5 h-5" />
              </button>
              <button
                onClick={exportToCSV}
                className="flex items-center gap-2 bg-white/20 hover:bg-white/30 px-3 py-2 rounded-lg transition-all"
              >
                <Download className="w-5 h-5" />
              </button>
            </div>
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 py-6">
        {activeView === 'dashboard' ? (
          <>
            {/* Stats Cards */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6 print:hidden">
              <div className="bg-gradient-to-br from-emerald-500 to-emerald-600 rounded-2xl p-6 text-white shadow-lg">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-emerald-100 text-sm mb-1">کل واریزی‌ها</p>
                    <p className="text-2xl font-bold">{formatToman(stats.totalDeposits)}</p>
                  </div>
                  <div className="bg-white/20 p-3 rounded-xl">
                    <TrendingUp className="w-8 h-8" />
                  </div>
                </div>
              </div>

              <div className="bg-gradient-to-br from-rose-500 to-rose-600 rounded-2xl p-6 text-white shadow-lg">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-rose-100 text-sm mb-1">کل هزینه‌ها</p>
                    <p className="text-2xl font-bold">{formatToman(stats.totalExpenses)}</p>
                  </div>
                  <div className="bg-white/20 p-3 rounded-xl">
                    <TrendingDown className="w-8 h-8" />
                  </div>
                </div>
              </div>

              <div className={`rounded-2xl p-6 text-white shadow-lg ${
                stats.balance >= 0 
                  ? 'bg-gradient-to-br from-blue-500 to-blue-600' 
                  : 'bg-gradient-to-br from-amber-500 to-amber-600'
              }`}>
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-white/80 text-sm mb-1">مانده موجودی</p>
                    <p className="text-2xl font-bold">{formatToman(Math.abs(stats.balance))}</p>
                  </div>
                  <div className="bg-white/20 p-3 rounded-xl">
                    <Wallet className="w-8 h-8" />
                  </div>
                </div>
              </div>

              <div className="bg-gradient-to-br from-violet-500 to-violet-600 rounded-2xl p-6 text-white shadow-lg">
                <div className="flex items-center justify-between">
                  <div>
                    <p className="text-violet-100 text-sm mb-1">تعداد تراکنش‌ها</p>
                    <p className="text-2xl font-bold">{toPersianNumber(transactions.length)}</p>
                  </div>
                  <div className="bg-white/20 p-3 rounded-xl">
                    <CreditCard className="w-8 h-8" />
                  </div>
                </div>
              </div>
            </div>

            {/* Bank Accounts */}
            <div className="bg-white rounded-2xl shadow-lg p-6 mb-6 print:hidden">
              <h2 className="text-xl font-bold text-slate-800 mb-4 flex items-center gap-2">
                <Building2 className="w-6 h-6 text-blue-600" />
                موجودی حساب‌ها
              </h2>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                {stats.bankBalances.map(account => (
                  <div key={account.id} className="border border-slate-200 rounded-xl p-4 hover:shadow-md transition-shadow">
                    <div className="flex items-center gap-3 mb-3">
                      <div 
                        className="w-14 h-14 rounded-xl flex items-center justify-center text-2xl"
                        style={{ backgroundColor: `${account.color}20` }}
                      >
                        {account.logo}
                      </div>
                      <div>
                        <h3 className="font-bold text-slate-800">{account.name}</h3>
                        <p className="text-sm text-slate-500">{account.owner}</p>
                      </div>
                    </div>
                    <div className="text-right">
                      <p className="text-2xl font-bold" style={{ color: account.color }}>
                        {formatToman(account.balance)}
                      </p>
                      <p className="text-sm text-slate-500">موجودی</p>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            {/* Charts */}
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6 print:hidden">
              {/* Weekly Comparison */}
              <div className="bg-white rounded-2xl shadow-lg p-6">
                <h3 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                  <BarChart3 className="w-5 h-5 text-blue-600" />
                  مقایسه هفتگی (تومان)
                </h3>
                <ResponsiveContainer width="100%" height={250}>
                  <BarChart data={weeklyData}>
                    <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                    <XAxis dataKey="date" tick={{ fill: '#64748b' }} />
                    <YAxis tick={{ fill: '#64748b' }} />
                    <Tooltip 
                      formatter={tooltipFormatter}
                      contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)' }}
                    />
                    <Bar dataKey="deposits" name="واریزی" fill="#10b981" radius={[8, 8, 0, 0]} />
                    <Bar dataKey="expenses" name="هزینه" fill="#f43f5e" radius={[8, 8, 0, 0]} />
                    <Legend />
                  </BarChart>
                </ResponsiveContainer>
              </div>

              {/* Expense Categories */}
              <div className="bg-white rounded-2xl shadow-lg p-6">
                <h3 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                  <PieChart className="w-5 h-5 text-purple-600" />
                  توزیع هزینه‌ها
                </h3>
                {categoryStats.length > 0 ? (
                  <ResponsiveContainer width="100%" height={250}>
                    <RePieChart>
                      <Pie
                        data={categoryStats}
                        cx="50%"
                        cy="50%"
                        innerRadius={60}
                        outerRadius={90}
                        paddingAngle={5}
                        dataKey="value"
                      >
                        {categoryStats.map((entry, index) => (
                          <Cell key={`cell-${index}`} fill={entry.color} />
                        ))}
                      </Pie>
                      <Tooltip 
                        formatter={pieTooltipFormatter}
                        contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)' }}
                      />
                      <Legend />
                    </RePieChart>
                  </ResponsiveContainer>
                ) : (
                  <div className="flex items-center justify-center h-[250px] text-slate-400">
                    داده‌ای موجود نیست
                  </div>
                )}
              </div>
            </div>

            {/* Category Weekly Breakdown */}
            <div className="bg-white rounded-2xl shadow-lg p-6 mb-6 print:hidden">
              <h3 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                <Filter className="w-5 h-5 text-orange-600" />
                هزینه‌های هفتگی به تفکیک دسته‌بندی (تومان)
              </h3>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={categoryWeeklyData}>
                  <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                  <XAxis dataKey="date" tick={{ fill: '#64748b' }} />
                  <YAxis tick={{ fill: '#64748b' }} />
                  <Tooltip 
                    formatter={tooltipFormatter}
                    contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)' }}
                  />
                  <Legend />
                  {expenseCategories.map(cat => (
                    <Bar 
                      key={cat.id}
                      dataKey={cat.id} 
                      name={cat.name} 
                      fill={cat.color} 
                      radius={[4, 4, 0, 0]}
                    />
                  ))}
                </BarChart>
              </ResponsiveContainer>
            </div>

            {/* Transactions Table */}
            <div className="bg-white rounded-2xl shadow-lg overflow-hidden print:shadow-none">
              <div className="p-6 border-b border-slate-200 print:hidden">
                <div className="flex flex-col md:flex-row md:items-center justify-between gap-4">
                  <h2 className="text-xl font-bold text-slate-800 flex items-center gap-2">
                    <CreditCard className="w-6 h-6 text-blue-600" />
                    لیست تراکنش‌ها
                  </h2>
                  
                  <div className="flex flex-wrap gap-3">
                    <div className="relative">
                      <Search className="w-5 h-5 text-slate-400 absolute right-3 top-1/2 -translate-y-1/2" />
                      <input
                        type="text"
                        placeholder="جستجو..."
                        value={searchTerm}
                        onChange={(e) => setSearchTerm(e.target.value)}
                        className="pl-4 pr-10 py-2 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 w-full md:w-64"
                      />
                    </div>
                    <select
                      value={filterType}
                      onChange={(e) => setFilterType(e.target.value as 'all' | 'deposit' | 'expense')}
                      className="px-4 py-2 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                    >
                      <option value="all">همه</option>
                      <option value="deposit">واریزی</option>
                      <option value="expense">هزینه</option>
                    </select>
                  </div>
                </div>
              </div>
              
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead className="bg-slate-50 border-b border-slate-200 print:bg-slate-100">
                    <tr>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">#</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">تاریخ</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">نوع</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">مبلغ</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">حساب</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">دسته‌بندی</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">شرح</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700 print:hidden">عملیات</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-100">
                    {filteredTransactions.map((transaction, idx) => {
                      const bank = bankAccounts.find(b => b.id === transaction.bankAccount);
                      const category = expenseCategories.find(c => c.id === transaction.category);
                      return (
                        <tr key={transaction.id} className="hover:bg-slate-50 transition-colors">
                          <td className="px-6 py-4 text-slate-500">
                            {toPersianNumber(idx + 1)}
                          </td>
                          <td className="px-6 py-4">
                            <div className="flex flex-col">
                              <span className="font-medium text-slate-800">
                                {toPersianNumber(transaction.date)}
                              </span>
                              <span className="text-xs text-slate-400">
                                {getDayName(transaction.date)}
                              </span>
                            </div>
                          </td>
                          <td className="px-6 py-4">
                            <span className={`inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-sm font-medium ${
                              transaction.type === 'deposit'
                                ? 'bg-emerald-100 text-emerald-700'
                                : 'bg-rose-100 text-rose-700'
                            }`}>
                              {transaction.type === 'deposit' ? (
                                <><ArrowDownRight className="w-4 h-4" /> واریزی</>
                              ) : (
                                <><ArrowUpRight className="w-4 h-4" /> هزینه</>
                              )}
                            </span>
                          </td>
                          <td className="px-6 py-4">
                            <span className={`font-bold text-lg ${
                              transaction.type === 'deposit' ? 'text-emerald-600' : 'text-rose-600'
                            }`}>
                              {transaction.type === 'deposit' ? '+' : '-'}
                              {formatToman(transaction.amount)}
                            </span>
                          </td>
                          <td className="px-6 py-4">
                            <div className="flex items-center gap-2">
                              <span className="text-2xl">{bank?.logo}</span>
                              <div>
                                <p className="font-medium text-slate-800">{bank?.name}</p>
                                <p className="text-xs text-slate-500">{bank?.owner}</p>
                              </div>
                            </div>
                          </td>
                          <td className="px-6 py-4">
                            {category ? (
                              <div className="flex items-center gap-2">
                                <div 
                                  className="w-3 h-3 rounded-full"
                                  style={{ backgroundColor: category.color }}
                                />
                                <span className="text-sm text-slate-700">{category.name}</span>
                              </div>
                            ) : (
                              <span className="text-slate-400">-</span>
                            )}
                          </td>
                          <td className="px-6 py-4 text-slate-700">
                            {transaction.description}
                          </td>
                          <td className="px-6 py-4 print:hidden">
                            <button
                              onClick={() => deleteTransaction(transaction.id)}
                              className="text-rose-500 hover:text-rose-700 p-2 hover:bg-rose-50 rounded-lg transition-colors"
                            >
                              <Trash2 className="w-5 h-5" />
                            </button>
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            </div>
          </>
        ) : (
          /* Monthly View */
          <div className="space-y-6 print:hidden">
            {/* Month Selector */}
            <div className="bg-white rounded-2xl shadow-lg p-6">
              <div className="flex items-center justify-between mb-6">
                <h2 className="text-xl font-bold text-slate-800 flex items-center gap-2">
                  <Calendar className="w-6 h-6 text-blue-600" />
                  نمای سالانه {toPersianNumber(currentYear)}
                </h2>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => setCurrentYear(y => y - 1)}
                    className="p-2 hover:bg-slate-100 rounded-lg transition-colors"
                  >
                    <ChevronRight className="w-5 h-5" />
                  </button>
                  <span className="text-lg font-bold">{toPersianNumber(currentYear)}</span>
                  <button
                    onClick={() => setCurrentYear(y => y + 1)}
                    className="p-2 hover:bg-slate-100 rounded-lg transition-colors"
                  >
                    <ChevronLeft className="w-5 h-5" />
                  </button>
                </div>
              </div>

              {/* Month Cards */}
              <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-3">
                {monthlyData.map((month, idx) => (
                  <button
                    key={idx}
                    onClick={() => setSelectedMonth(idx)}
                    className={`p-4 rounded-xl border-2 transition-all text-right ${
                      selectedMonth === idx
                        ? 'border-blue-500 bg-blue-50'
                        : 'border-slate-200 hover:border-blue-300'
                    }`}
                  >
                    <p className={`font-bold mb-2 ${
                      selectedMonth === idx ? 'text-blue-700' : 'text-slate-700'
                    }`}>
                      {month.name}
                    </p>
                    <div className="space-y-1 text-xs">
                      <p className="text-emerald-600">
                        واریز: {formatNumber(month.deposits)} ت
                      </p>
                      <p className="text-rose-600">
                        هزینه: {formatNumber(month.expenses)} ت
                      </p>
                    </div>
                  </button>
                ))}
              </div>
            </div>

            {/* Selected Month Details */}
            <div className="bg-gradient-to-br from-blue-600 to-blue-700 rounded-2xl shadow-lg p-6 text-white">
              <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
                <Calendar className="w-6 h-6" />
                جزئیات {months[selectedMonth]} {toPersianNumber(currentYear)}
              </h3>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div className="bg-white/10 rounded-xl p-4 backdrop-blur-sm">
                  <p className="text-blue-100 text-sm mb-1">کل واریزی‌ها</p>
                  <p className="text-2xl font-bold">{formatToman(selectedMonthData.deposits)}</p>
                </div>
                <div className="bg-white/10 rounded-xl p-4 backdrop-blur-sm">
                  <p className="text-blue-100 text-sm mb-1">کل هزینه‌ها</p>
                  <p className="text-2xl font-bold">{formatToman(selectedMonthData.expenses)}</p>
                </div>
                <div className="bg-white/10 rounded-xl p-4 backdrop-blur-sm">
                  <p className="text-blue-100 text-sm mb-1">مانده</p>
                  <p className={`text-2xl font-bold ${
                    selectedMonthData.deposits - selectedMonthData.expenses >= 0 
                      ? '' : 'text-amber-300'
                  }`}>
                    {formatToman(Math.abs(selectedMonthData.deposits - selectedMonthData.expenses))}
                  </p>
                </div>
              </div>
            </div>

            {/* Monthly Charts */}
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <div className="bg-white rounded-2xl shadow-lg p-6">
                <h3 className="text-lg font-bold text-slate-800 mb-4">
                  روند سالانه (تومان)
                </h3>
                <ResponsiveContainer width="100%" height={250}>
                  <LineChart data={monthlyData}>
                    <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                    <XAxis dataKey="name" tick={{ fill: '#64748b' }} />
                    <YAxis tick={{ fill: '#64748b' }} />
                    <Tooltip 
                      formatter={tooltipFormatter}
                      contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)' }}
                    />
                    <Line type="monotone" dataKey="deposits" name="واریزی" stroke="#10b981" strokeWidth={3} />
                    <Line type="monotone" dataKey="expenses" name="هزینه" stroke="#f43f5e" strokeWidth={3} />
                    <Legend />
                  </LineChart>
                </ResponsiveContainer>
              </div>

              <div className="bg-white rounded-2xl shadow-lg p-6">
                <h3 className="text-lg font-bold text-slate-800 mb-4">
                  دسته‌بندی هزینه‌های {months[selectedMonth]}
                </h3>
                {selectedMonthData.categoryData.length > 0 ? (
                  <ResponsiveContainer width="100%" height={250}>
                    <RePieChart>
                      <Pie
                        data={selectedMonthData.categoryData}
                        cx="50%"
                        cy="50%"
                        innerRadius={60}
                        outerRadius={90}
                        paddingAngle={5}
                        dataKey="value"
                      >
                        {selectedMonthData.categoryData.map((entry, index) => (
                          <Cell key={`cell-${index}`} fill={entry.color} />
                        ))}
                      </Pie>
                      <Tooltip 
                        formatter={pieTooltipFormatter}
                        contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 4px 6px -1px rgba(0, 0, 0, 0.1)' }}
                      />
                      <Legend />
                    </RePieChart>
                  </ResponsiveContainer>
                ) : (
                  <div className="flex items-center justify-center h-[250px] text-slate-400">
                    داده‌ای موجود نیست
                  </div>
                )}
              </div>
            </div>

            {/* Monthly Transactions */}
            <div className="bg-white rounded-2xl shadow-lg overflow-hidden">
              <div className="p-6 border-b border-slate-200">
                <h3 className="text-lg font-bold text-slate-800">
                  تراکنش‌های {months[selectedMonth]}
                </h3>
              </div>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead className="bg-slate-50">
                    <tr>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">تاریخ</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">نوع</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">مبلغ</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">دسته‌بندی</th>
                      <th className="px-6 py-4 text-right font-bold text-slate-700">شرح</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-100">
                    {selectedMonthData.transactions.length > 0 ? (
                      selectedMonthData.transactions.map((transaction) => {
                        const category = expenseCategories.find(c => c.id === transaction.category);
                        return (
                          <tr key={transaction.id} className="hover:bg-slate-50">
                            <td className="px-6 py-4">
                              {toPersianNumber(transaction.date)}
                            </td>
                            <td className="px-6 py-4">
                              <span className={`inline-flex items-center gap-1 px-3 py-1 rounded-full text-sm ${
                                transaction.type === 'deposit'
                                  ? 'bg-emerald-100 text-emerald-700'
                                  : 'bg-rose-100 text-rose-700'
                              }`}>
                                {transaction.type === 'deposit' ? 'واریزی' : 'هزینه'}
                              </span>
                            </td>
                            <td className="px-6 py-4 font-bold">
                              {formatToman(transaction.amount)}
                            </td>
                            <td className="px-6 py-4">
                              {category ? (
                                <div className="flex items-center gap-2">
                                  <div 
                                    className="w-3 h-3 rounded-full"
                                    style={{ backgroundColor: category.color }}
                                  />
                                  {category.name}
                                </div>
                              ) : '-'}
                            </td>
                            <td className="px-6 py-4 text-slate-600">
                              {transaction.description}
                            </td>
                          </tr>
                        );
                      })
                    ) : (
                      <tr>
                        <td colSpan={5} className="px-6 py-8 text-center text-slate-400">
                          تراکنشی در این ماه ثبت نشده است
                        </td>
                      </tr>
                    )}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        )}
      </main>

      {/* Add Transaction Modal */}
      {showForm && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50 print:hidden">
          <div className="bg-white rounded-2xl shadow-2xl w-full max-w-lg max-h-[90vh] overflow-y-auto">
            <div className="sticky top-0 bg-white border-b border-slate-200 p-6 flex items-center justify-between">
              <h2 className="text-xl font-bold text-slate-800 flex items-center gap-2">
                <Plus className="w-6 h-6 text-blue-600" />
                ثبت تراکنش جدید
              </h2>
              <button
                onClick={() => setShowForm(false)}
                className="p-2 hover:bg-slate-100 rounded-lg transition-colors"
              >
                <X className="w-5 h-5 text-slate-500" />
              </button>
            </div>
            
            <form onSubmit={handleSubmit} className="p-6 space-y-5">
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-2">
                    نوع تراکنش
                  </label>
                  <div className="grid grid-cols-2 gap-2">
                    <button
                      type="button"
                      onClick={() => setFormData({ ...formData, type: 'deposit' })}
                      className={`flex items-center justify-center gap-2 py-3 px-4 rounded-xl border-2 transition-all ${
                        formData.type === 'deposit'
                          ? 'border-emerald-500 bg-emerald-50 text-emerald-700'
                          : 'border-slate-200 hover:border-emerald-300'
                      }`}
                    >
                      <TrendingUp className="w-5 h-5" />
                      واریزی
                    </button>
                    <button
                      type="button"
                      onClick={() => setFormData({ ...formData, type: 'expense' })}
                      className={`flex items-center justify-center gap-2 py-3 px-4 rounded-xl border-2 transition-all ${
                        formData.type === 'expense'
                          ? 'border-rose-500 bg-rose-50 text-rose-700'
                          : 'border-slate-200 hover:border-rose-300'
                      }`}
                    >
                      <TrendingDown className="w-5 h-5" />
                      هزینه
                    </button>
                  </div>
                </div>

                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-2">
                    تاریخ (میلادی)
                  </label>
                  <div className="relative">
                    <Calendar className="w-5 h-5 text-slate-400 absolute right-3 top-1/2 -translate-y-1/2" />
                    <input
                      type="date"
                      value={formData.date}
                      onChange={(e) => setFormData({ ...formData, date: e.target.value })}
                      className="w-full pl-4 pr-10 py-3 border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500"
                      required
                    />
                  </div>
                </div>
              </div>

              <div>
                <label className="block text-sm font-medium text-slate-700 mb-2">
                  مبلغ (تومان)
                </label>
                <div className="relative">
                  <input
                    type="number"
                    value={formData.amount}
                    onChange={(e) => setFormData({ ...formData, amount: e.target.value })}
                    className="w-full pl-20 pr-4 py-3 border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500"
                    placeholder="مثال: 500000"
                    required
                  />
                  <span className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-500 text-sm">
                    تومان
                  </span>
                </div>
                {formData.amount && (
                  <p className="text-sm text-slate-500 mt-1">
                    {toPersianNumber(parseInt(formData.amount).toLocaleString())} تومان
                  </p>
                )}
              </div>

              <div>
                <label className="block text-sm font-medium text-slate-700 mb-2">
                  حساب بانکی
                </label>
                <div className="grid grid-cols-1 gap-2">
                  {bankAccounts.map(account => (
                    <button
                      key={account.id}
                      type="button"
                      onClick={() => setFormData({ ...formData, bankAccount: account.id })}
                      className={`flex items-center gap-3 p-4 rounded-xl border-2 transition-all text-right ${
                        formData.bankAccount === account.id
                          ? 'border-blue-500 bg-blue-50'
                          : 'border-slate-200 hover:border-blue-300'
                      }`}
                    >
                      <span className="text-3xl">{account.logo}</span>
                      <div className="flex-1">
                        <p className="font-bold text-slate-800">{account.name}</p>
                        <p className="text-sm text-slate-500">{account.owner}</p>
                      </div>
                      {formData.bankAccount === account.id && (
                        <div className="w-5 h-5 rounded-full bg-blue-500 flex items-center justify-center">
                          <div className="w-2 h-2 bg-white rounded-full" />
                        </div>
                      )}
                    </button>
                  ))}
                </div>
              </div>

              {formData.type === 'expense' && (
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-2">
                    دسته‌بندی هزینه
                  </label>
                  <div className="grid grid-cols-2 gap-2">
                    {expenseCategories.map(cat => {
                      const Icon = cat.icon;
                      return (
                        <button
                          key={cat.id}
                          type="button"
                          onClick={() => setFormData({ ...formData, category: cat.id })}
                          className={`flex items-center gap-2 p-3 rounded-xl border-2 transition-all ${
                            formData.category === cat.id
                              ? 'border-orange-500 bg-orange-50'
                              : 'border-slate-200 hover:border-orange-300'
                          }`}
                        >
                          <Icon className="w-5 h-5" style={{ color: cat.color }} />
                          <span className="text-sm">{cat.name}</span>
                        </button>
                      );
                    })}
                  </div>
                </div>
              )}

              <div>
                <label className="block text-sm font-medium text-slate-700 mb-2">
                  شرح تراکنش
                </label>
                <textarea
                  value={formData.description}
                  onChange={(e) => setFormData({ ...formData, description: e.target.value })}
                  className="w-full px-4 py-3 border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"
                  rows={3}
                  placeholder="توضیحات تراکنش..."
                  required
                />
              </div>

              <div className="flex gap-3 pt-4">
                <button
                  type="button"
                  onClick={() => setShowForm(false)}
                  className="flex-1 py-3 px-4 border border-slate-300 rounded-xl text-slate-700 font-medium hover:bg-slate-50 transition-colors"
                >
                  انصراف
                </button>
                <button
                  type="submit"
                  className="flex-1 py-3 px-4 bg-gradient-to-r from-blue-600 to-blue-700 text-white rounded-xl font-medium hover:from-blue-700 hover:to-blue-800 transition-all shadow-lg"
                >
                  ثبت تراکنش
                </button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
}  REPO     PREDICATE_TYPE                  WORKFLOW
  cli/cli  https://slsa.dev/provenance/v1  .github/workflows/deployment.yml@refs/heads/trunk
  ```

- **Option 2: Using Sigstore [`cosign`](https://github.com/sigstore/cosign):**

  To perform this, download the [attestation](https://github.com/cli/cli/attestations) for the downloaded release and use cosign to verify the authenticity of the downloaded release:

  ```shell
  $ cosign verify-blob-attestation --bundle cli-cli-attestation-3120304.sigstore.json \
        --new-bundle-format \
        --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
        --certificate-identity="https://github.com/cli/cli/.github/workflows/deployment.yml@refs/heads/trunk" \
        gh_2.62.0_macOS_arm64.zip
  Verified OK
  ```

## Comparison with hub

For many years, [hub](https://github.com/github/hub) was the unofficial GitHub CLI tool. `gh` is a new project that helps us explore
what an official GitHub CLI tool can look like with a fundamentally different design. While both
tools bring GitHub to the terminal, `hub` behaves as a proxy to `git`, and `gh` is a standalone
tool. Check out our [more detailed explanation](docs/gh-vs-hub.md) to learn more.

[releases page]: https://github.com/cli/cli/releases/latest
