import React, { useRef } from 'react';
import { Upload, X, Image } from 'lucide-react';
import { Button } from "@/components/ui/button";
import { base44 } from '@/api/base44Client';
import { motion, AnimatePresence } from 'framer-motion';

export default function LogoUploader({ logoUrl, onLogoChange }) {
  const fileInputRef = useRef(null);

  const handleUpload = async (e) => {
    const file = e.target.files[0];
    if (file) {
      const { file_url } = await base44.integrations.Core.UploadFile({ file });
      onLogoChange(file_url);
    }
  };

  const handleRemove = () => {
    onLogoChange('');
  };

  return (
    <div className="relative">
      <AnimatePresence mode="wait">
        {logoUrl ? (
          <motion.div
            initial={{ opacity: 0, scale: 0.9 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.9 }}
            className="relative group"
          >
            <img 
              src={logoUrl} 
              alt="Logo" 
              className="h-16 w-16 rounded-xl object-cover border border-slate-200 shadow-sm"
            />
            <Button
              size="icon"
              variant="destructive"
              onClick={handleRemove}
              className="absolute -top-2 -right-2 h-5 w-5 rounded-full opacity-0 group-hover:opacity-100 transition-opacity"
            >
              <X className="h-3 w-3" />
            </Button>
          </motion.div>
        ) : (
          <motion.div
            initial={{ opacity: 0, scale: 0.9 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.9 }}
          >
            <Button
              variant="outline"
              size="icon"
              onClick={() => fileInputRef.current?.click()}
              className="h-16 w-16 rounded-xl border-dashed border-2 border-slate-300 hover:border-indigo-400 hover:bg-indigo-50 transition-all"
            >
              <Image className="h-5 w-5 text-slate-400" />
            </Button>
          </motion.div>
        )}
      </AnimatePresence>
      <input
        ref={fileInputRef}
        type="file"
        accept="image/*"
        onChange={handleUpload}
        className="hidden"
      />
    </div>
  );
}