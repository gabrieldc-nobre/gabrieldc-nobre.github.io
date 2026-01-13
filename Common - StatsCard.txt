import React from 'react';
import { motion } from 'framer-motion';
import { cn } from "@/lib/utils";

export default function StatsCard({ 
  title, 
  value, 
  icon: Icon, 
  color = 'indigo',
  onClick,
  active = false 
}) {
  const colorClasses = {
    indigo: {
      bg: 'bg-indigo-50',
      icon: 'bg-indigo-100 text-indigo-600',
      border: 'border-indigo-200',
      text: 'text-indigo-600'
    },
    emerald: {
      bg: 'bg-emerald-50',
      icon: 'bg-emerald-100 text-emerald-600',
      border: 'border-emerald-200',
      text: 'text-emerald-600'
    },
    amber: {
      bg: 'bg-amber-50',
      icon: 'bg-amber-100 text-amber-600',
      border: 'border-amber-200',
      text: 'text-amber-600'
    },
    slate: {
      bg: 'bg-slate-50',
      icon: 'bg-slate-100 text-slate-600',
      border: 'border-slate-200',
      text: 'text-slate-600'
    },
    rose: {
      bg: 'bg-rose-50',
      icon: 'bg-rose-100 text-rose-600',
      border: 'border-rose-200',
      text: 'text-rose-600'
    }
  };

  const colors = colorClasses[color] || colorClasses.indigo;

  return (
    <motion.div
      whileHover={{ scale: 1.02, y: -2 }}
      whileTap={{ scale: 0.98 }}
      onClick={onClick}
      className={cn(
        "p-5 rounded-2xl border-2 cursor-pointer transition-all",
        active ? `${colors.bg} ${colors.border}` : 'bg-white border-transparent shadow-sm hover:shadow-md'
      )}
    >
      <div className="flex items-start justify-between">
        <div>
          <p className="text-sm font-medium text-slate-500 mb-1">{title}</p>
          <p className={cn("text-3xl font-bold", colors.text)}>{value}</p>
        </div>
        <div className={cn("p-3 rounded-xl", colors.icon)}>
          <Icon className="h-5 w-5" />
        </div>
      </div>
    </motion.div>
  );
}